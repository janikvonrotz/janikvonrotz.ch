---
title: "Apollo Graphql integration testing in practice"
slug: apollo-graphql-integration-testing-in-practice
date: 2019-12-05T10:16:19+01:00
categories:
 - Software development
tags:
 - integration
 - testing
 - graphql
 - apollo
images:
 - /images/intertwined.jpg
---

The [Apollo Graphql documention](https://www.apollographql.com/docs) offers a pretty comprehensive guide on how to [test your Graphql API](https://www.apollographql.com/docs/apollo-server/testing/testing). But sure its different once you implement tests for your non generic project. That is why I wrote this post. First I would like to introduce you to integration tests and how they are different from the other testing levels. And then I'll give you a hands on experience for writing integration tests for Apollo Graphql with [Ava](https://github.com/avajs/ava).

## Testing levels

As promised first we will have a look at the different testing levels. For this I created this beautiful visualization:

![](/images/Testing levels.png)

On the right you'll see a distributed software system and wrapped in boxes you'll see what type of test type focuses on which components of the software system.

## Unit Tests

When considering the entire software system and functions that can be tested, unit tests are the smallest test methods. Unit tests check methods of classes and objects of the source code. They are used to ensure that the smallest possible testable functions behaves correctly. The overall function of the software system is neglected. The highest possible test coverage is desired here. The effort for implementation is low.

## Integration Tests

Integration tests are used to check whether the application is running correctly on the server. Thus the server environment is also included in the test.  The test coverage is lower compared to the unit tests. Integration tests show how interoperable the application is.

## End to End Tests

An application connects to various services and databases. End to end tests are used to check communication with surrounding systems. The implementation effort for end-to-end tests is high. The test coverage is low. With end-to-end tests, statements can be made about the overall behavior of the software system.

## UI Tests

UI tests are the most complex form of tests. They simulate the behavior of a user. With the help of automation tools, the same actions of a user are executed in the client application. The implementation of UI tests is extremely time-consuming and is known for continuous need of repair. Every change to the software system affects the UI tests.

# Integration tests with Apollo Graphql

To showcase how integration tests are implemented for an Apollo Graphql API we need a schema, mutation, queries and resolvers. Here is an Graphql type example:

**schema.js**

```gql
...
type Tenant {
  id: String!
  name: String!
  assigned_users: [User]
  assigned_category: Category

  created: Date
  created_by: User!
  updated: Date
  updated_by: User!
}
...
```

The API provides these queries and mutations.

```gql
...
type Query {
  tenants: [Tenant] @hasRole(roles: [ADMIN])
  tenant(id: String): Tenant @hasRole(roles: [ADMIN])
}
...
type Mutation {
  createTenant(name: String!): Tenant @hasRole(roles: [ADMIN])
  updateTenant(
    id: String!
    name: String
    assigned_users: [String]
  ): Response @hasRole(roles: [ADMIN])
  deleteTenant(id: String!): Response @hasRole(roles: [ADMIN])
  assignTenant(
    id: String!,
    user: String!
  ): Response @hasRole(roles: [ADMIN])
}
...
```

As you can see there are other types and directives in the example schema. For this tutorial we focus on the tenant type, ignore the others.

Running integration tests means I want know wether the resolvers of the queries and mutations work correctly.

Therefore we have to start the server, bootstrap the database, run the tests and clean everysthing.

In your codebase test code is separated from the functional code. In Ava every test case has its own standalone file for running the tests.

To test resolvers I have created this file:

**resolvers.test.js**

I am using the official Apollo server testing package to setup the test server.

*Imports:*
```js
const { createTestClient } = require('apollo-server-testing')
const { ApolloServer, gql } = require('apollo-server-micro')
const typeDefs = require('./schema')
const directives = require('./directives')
const { merge } = require('lodash')
const tenantResolvers = require('./resolvers-tenant')
const test = require('ava')
const { ObjectId } = require('mongodb')
...
```

Ava is my test runner. The `ObjectId` is used to validate if the resolvers actually returns a correct mongo object id.

*Environment config:*

```js
...
// Load environment configuration
require('dotenv').config({ path: `${__dirname}/.env.${process.env.NODE_ENV}` })
...
```

Tests run in a separate environment. Env variables are loaded from the `.env.test` file.

*Context and server initialization:*

```js
...
// Initialize Apollo server
var context = { user: { id: 1, email: 'admin@labtrail.app', role: 'ADMIN', tenant: 1 } }
const server = new ApolloServer({
  typeDefs,
  resolvers: merge(
    tenantResolvers
  ),
  context: () => (context),
  schemaDirectives: directives
})
const { query, mutate } = createTestClient(server)
...
```

Here the Apollo Server is initialized with a context mock. The context mock ensure that the tests have the privileges required to run the queries and mutations.

*First test:*

```js
...
// Share context between tests
var result = {}

test.serial('Create tenant Acme', async t => {
  const CREATE_TENANT = gql`
  mutation createTenant( $name: String!) {
    createTenant(name: $name) {
      id
    }
  }
  `
  result = merge(result, await mutate({
    mutation: CREATE_TENANT,
    variables: { name: 'Acme' }
  }))
  t.assert(ObjectId.isValid(result.data.createTenant.id))
})
...
```

In our test scenario tests depend on each other. They must be executed serially. Every test has a description, runs a query or mutation and asserts the result at the end of the async test function.

*More tests:*

```js
...
test.serial('Get tenant Acme by Id', async t => {
  const GET_TENANT = gql`
  query tenant($id: String) {
    tenant(id: $id) {
      id
      name
    }
  }
  `
  result = merge(result, await query({
    query: GET_TENANT,
    variables: { id: result.data.createTenant.id }
  }))
  t.is(result.data.tenant.name, 'Acme')
})

test.serial('Mutate tenant Acme to AcmeX', async t => {
  const UPDATE_TENANT = gql`
  mutation updateTenant($id: String!, $name: String) {
    updateTenant(id: $id, name: $name) {
      success
    }
  }
  `
  result = merge(result, await mutate({
    mutation: UPDATE_TENANT,
    variables: { id: result.data.createTenant.id, name: 'AcmeX' }
  }))
  t.assert(result.data.updateTenant.success)
})

test.serial('Delete tenant Acme by Id', async t => {
  const DELETE_TENANT = gql`
  mutation deleteTenant( $id: String!) {
    deleteTenant(id: $id) {
      success
    }
  }
  `
  result = merge(result, await mutate({
    mutation: DELETE_TENANT,
    variables: { id: result.data.createTenant.id }
  }))
  t.assert(result.data.deleteTenant.success)
})
```

The tests shown here match queries and mutations of the schema. For every type and set of queries and mutations there is separate test file. Ava runs them concurrently, but ensures that the single tests cases are executed serially.

Running the tests is simple. I created a script task to run Ava with yarn.

**package.json**

```json
  ...
  "scripts": {
    ...
    "test": "ava"
  },
  ...
```

And running `yarn test` will do the job.

For more details on the Ava checkout the [documentation](https://github.com/avajs/ava/tree/master/docs).

Hope this tutorial gave you an idea about testing and how to apply it to Apollo Graphql 😊