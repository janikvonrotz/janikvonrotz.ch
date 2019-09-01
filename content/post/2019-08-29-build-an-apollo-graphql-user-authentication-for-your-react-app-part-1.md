---
title: "Build an Apollo Graphql user authentication for your React app - part 1"
slug: build-an-apollo-graphql-user-authentication-for-your-react-app-part-1
date: 2019-08-27T18:35:12+02:00
categories:
 - JavaScript development
tags:
 - apollo
 - graphql
 - authentication
 - react
 - json web token
 - directive
images:
 - /images/metal network.jpg
---

I am currently building an [Apollo Graphql](https://www.apollographql.com/docs/apollo-server/) API and a [React](https://reactjs.org/) web application. The application requires a user authentication functionality in order to enforce access restrictions on the Graphql endpoint. Apollo Graphql does not provide an out-of-the-box-solution and therefore I would like to present my solution.
<!--more-->

This post requires some basic knowledge about building APIs with Apollo and later on combining the Apollo Client with React. First we are going to define the user schema and implement the resolvers. For the authentication mechanism we are going to implement a query that expects user credentials and returns a [JSON Web Token](https://jwt.io/) as response. This token will be used by the React app and passed as an [Bearer Authorization header](https://swagger.io/docs/specification/authentication/bearer-authentication/) to every sequentially API call. The Apollo server verifies the token and restricts access on certain calls if an invalid token has been provided. The access restriction is enforced on schema field level using a [custom directive](https://www.apollographql.com/docs/graphql-tools/schema-directives/).

## Files

My Apollo server implementation consists of the following files:

* `index.js`: Creates the Apollo server
* `schema.js`: Contains Graphql type definitions
* `resolvers.js`: Schema resolvers for data transformation and persistence
* `context.js`: Verifies the JWT token
* `directives.js`: Custom directives for the schema

**Note:** This guide does not cover how user data is stored and retrieved. Comments in the code snippets will point out where to implement these functionalities. In short this guide can be read regardless of the storage solution.

## Schema

Let us start with the type definitions. I have split the `schema.js` file into several parts.

**schema.js**

```txt
const { gql } = require('apollo-server-micro')

// GraphQL schema
const typeDefs = gql`

directive @isAuthenticated on FIELD_DEFINITION

scalar Date
```

The `Date` scalar is custom type, ignore it for now.
The `isAuthenticated` directive will be discussed later.

Next follows the user type.

```txt
type User {
  id: String!
  email: String!
  password: String!
  firstname: String!
  lastname: String!

  created: Date
  created_by: String!
  updated: Date
  updated_by: String!
}
```

Then the token and default response.

```txt
type Token {
  token: String!
}

type Response {
  success: Boolean!
  message: String
}
```

Queries are restricted by the custom directive.

```txt
type Query {
  currentUser: User @isAuthenticated
  users: [User] @isAuthenticated
  loginUser(email: String!, password: String!): Token
}
```

And here are the mutation we need to actually create and manipulate a user.

```txt
type Mutation {
  createUser(email: String!, password: String!, firstname: String!, lastname: String!): User
  updateUser(id: String!, email: String, password: String, firstname: String, lastname: String): Response
  deleteUser(id: String!): Response
}
`

module.exports = typeDefs
```

Put everything together and you got the schema definitions.

## Resolvers

In the resolvers file we map schema types and fields with functions that return the actual data. You can think of it as actually implementing the schema.

**resolvers.js**

```js
const { GraphQLScalarType } = require('graphql')
const { AuthenticationError, ForbiddenError } = require('apollo-server-micro')
const bcrypt = require('bcrypt')
const jwt = require('jsonwebtoken')
const { Kind } = require('graphql/language')

// Hash configuration
const BCRYPT_ROUNDS = 12

// Resolve GraphQL queries, mutations and graph paths
const resolvers = {
  Query: {
    users: async (obj, args, context) => {
      const user // -> Access data layer and get user data
      return user
    },
    currentUser: async (obj, args, context) => {
      const user // -> Access data layer and get user data
      return user
    },
    loginUser: async (obj, args, context) => {
      // Find user by email and password
      const user // -> Access data layer and get user data

      // Compare hash
      if(user && await bcrypt.compare(args.password, user.password)) {
        // Generate and return JWT token
        const token = jwt.sign({ email: user.email, name: (user.firstname + ' ' + user.lastname) }, process.env.JWT_SECRET )
        return { token: token }
      } else {
        // Throw authentication error
        throw new AuthenticationError('Login failed.')
      }
    }
  },
  Mutation: {
    createUser: async (obj, args, context) => {
      // Check if user already exists
      const user // -> Access data layer and get user data
      if (user) {
        throw new ForbiddenError('User already exists.')
      }

      // Hash password
      args.password = await bcrypt.hash(args.password, BCRYPT_ROUNDS)
      args.created = new Date()
      args.created_by = context.name || "system"
      const user // -> Access data layer and store user data
      return user
    },
    updateUser: async (obj, args, context) => {
      args.updated = new Date()
      args.updated_by = context.name || "system"

      // Hash password if provided
      if(args.password){
        args.password = await bcrypt.hash(args.password, BCRYPT_ROUNDS)
      }

      // -> Access data layer and update user data
      return { 
        success: true
      }
    },
    deleteUser: async (obj, args, context) => {
      // -> Access data layer and delete user data
      return { 
        success: true
      }
    }
  },
  User: {
    // Hide password hash
    password() {
      return ''
    }
  }
}

module.exports = resolvers
```

The most important part of the resolver files are the bcrypt and jwt method calls. If a user is created the plaintext password is hashed and stored in the database. If the `loginUser` query is called the user is retrieved and the password hash compared. If successful a JWT token is generated and returned.

## Context

We assume that during the login process in the client app the Apollo client calls the `loginUser` query, retrieves token and stores it in the local storage. If set the token is passed in the Bearer Authentication header with each API call. On the Apollo server side the token must be verified and its claims added to the Apollo context.

So what is the context? To put it simply the context is an object that is available to all resolvers. From the context user claims can be accessed and e.g. passed to a database query.

**context.js**

```js
const { AuthenticationError } = require('apollo-server-micro')
const jwt = require('jsonwebtoken')

const context = ({ req }) => {
  // Get the user token from the headers
  let token = req.headers.authorization ? req.headers.authorization.split(' ')[1] : null

  // Verify token if available
  if (token) {
    try {
      token = jwt.verify(token, process.env.JWT_SECRET)
    } catch (error) {
      throw new AuthenticationError(
        'Authentication token is invalid, please log in.'
      )
    }
  }

  return {
    email: token ? token.email : null,
    name: token ? token.name : null
  }
}

module.exports = context
```

If the token is set in the header and has been verified, the two claims `email` and `name` are then added to the context.

## Directive

As I mentioned we enforce schema field access with a custom directive. In the `isAuthenticated` directive we simply check if the `email` property is set in the context object.

```js
const { SchemaDirectiveVisitor, ForbiddenError } = require('apollo-server-micro')
const { defaultFieldResolver } = require('graphql')

// Custom directive to check if user is authenicated
class isAuthenticated extends SchemaDirectiveVisitor {
  // Field definition directive
  visitFieldDefinition (field) {
    // Get field resolver
    const { resolve = defaultFieldResolver } = field

    field.resolve = async function (...args) {
      // Check if user email is in context
      if (!args[2].email) {
        throw new ForbiddenError('You are not authorized for this ressource.')
      }

      // Resolve field
      return resolve.apply(this, args)
    }
  }
}

module.exports = { isAuthenticated: isAuthenticated }
```

Whenever a field declared with the `isAuthenticated` directive is queried, the directive will also ensure the user has a valid token.

## Server

In the `index.js` we stitch everything together.

```js
const { ApolloServer } = require('apollo-server-micro')
const cors = require('micro-cors')()
const typeDefs = require('./schema')
const resolvers = require('./resolvers')
const context = require('./context')
const directives = require('./directives')

// Initialize Apollo server
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: context,
  introspection: true,
  playground: true,
  schemaDirectives: directives
})

// Export server as handler
module.exports = cors((req, res) => {
  // Workaround abort on Options method
  if (req.method === 'OPTIONS') {
    res.end()
    return
  }
  return server.createHandler({ path: '/api' })(req, res)
})
```

To those who noticed that I import `apollo-server-micro` instead of `apollo-server`, it is because I am deploying my apps as serverless functions with [Zeit Now](https://zeit.co/home) using their [HTTP microservice](https://github.com/zeit/micro).

## Test

Start the Apollo server and open the Graphql playground.

Create a user with the `createUser` mutation.

```txt
mutation { 
  createUser(
    email: "login@example.com", 
    password: "password",
    firstname: "Login",
    lastname: "Example"
  ) { id } }
```

Login with this user.

```txt
{ loginUser(email: "login@example.org", password: "password") {
  token
} }
```

Copy the token in to the HTTP header field.

```json
{
    "Authorization": "Bearer YOUR_TOKEN_HERE"
}
```

Query the user info.

```txt
{ currentUser {
  firstname
  lastname
  email
  password
  id
} }
```

What the final query should look like:

![](/images/apollo-graphql-playground-query-user-info.png)

If this worked you have reached the end of part 1 😊.

## Next

In part 2 (to be linked soon) I will show how we implement the user authentication mechanism in a React app.

## Additions

I try to keep my guides as slick as possible and therefore intentionally do not cover certain parts. But at least I wanna let know you which parts I ignored.

In order to provide a full user accounting module of course more queries are required. You would at least have to implement the following ones:

* signupUser: Public query to create user account.
* verifyUserEmail: Query to verify user account.
* resetUserPassword: Query to initiate password reset.

If you have any questions, feel free to ask below 👋.

## Updates

*2019/08/30:* Updated all code snippets after applying JavaScript Standard Style guide. Removed unnecessary await statements.