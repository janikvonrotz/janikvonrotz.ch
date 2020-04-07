---
title: "Build an Apollo Graphql user authentication for your React app - part 3"
slug: build-an-apollo-graphql-user-authentication-for-your-react-app-part-3
date: 2019-09-26T10:31:27+02:00
categories:
 - JavaScript development
tags:
 - apollo
 - graphql
 - authentication
 - react
 - json web token
 - directive
 - authorization
images:
 - /images/metal network.jpg
---

This is the final post of my GraphQL Auth series. Before reading this post checkout [post 1](/2019/08/27/build-an-apollo-graphql-user-authentication-for-your-react-app-part-1) and [post 2](/2019/08/29/build-an-apollo-graphql-user-authentication-for-your-react-app-part-2).

As mentioned in my last post we need to polish our authentication solution. First we wanna ensure that the JWT token expires. Second, I think the `isAuthenticated` directive is insufficient for proper permission management on our types, queries and mutations. We need a role based solution. While the first point is simple to implement, the second is more complex and definitely requires walking through the previous posts.
<!--more-->

## Files

We will update the following files of our GraphQL server:

* `schema.js`: Type, query, mutation and directive definitions.
* `context.js`: Extracts claims from JWT token.
* `directives.js`: Directive implementations.
* `resolvers.js`: Resolve methods for schema.

## Schema

Update the GraphQL schema with a role direcive.

**schema.js**

```js
const { gql } = require('apollo-server-micro')

// GraphQL schema
const typeDefs = gql`
...

directive @hasRole(roles: [Role!]) on FIELD_DEFINITION

enum Role {
  ADMIN
  USER
}

...

type User {
  id: String!
  email: String!
  password: String!
  firstname: String!
  lastname: String!
  name: String
  role: Role!

  created: Date
  created_by: String!
  updated: Date
  updated_by: String!
}

type Query {
  currentUser: User @isAuthenticated
  users: [User] @hasRole(roles: [ADMIN])
}

type Mutation {
  createUser(email: String!, password: String!, firstname: String!, lastname: String!, role: Role): User @hasRole(roles: [ADMIN])
  updateUser(id: String!, email: String, password: String, firstname: String, lastname: String, role: Role): Response @hasRole(roles: [ADMIN])
  deleteUser(id: String!): Response @hasRole(roles: [ADMIN])
}
`
module.exports = typeDefs
```

In addition to our `isAuthenticated` there is now a `hasRole` directive. This directive accepts a list of roles and only grants user equipped with such role access or execution rights.

The `User` type has a new property `role`. It is now possible to assign a role from the `Role` enum to the user.

## Context

In the GraphQL context the JWT token is verified and the user details are retrieved from the store.

**context.js**

```js
...

  // Verify token if available
  if (token) {
    try {
      token = jwt.verify(token, process.env.JWT_SECRET)

      // Get user from database
      user = await (await usersCollection()).findOne({ email: token.email })

    } catch (error) {
      throw new AuthenticationError(
        'Authentication token is invalid, please log in.'
      )
    }
  }

  return {
    email: token ? token.email : null,
    name: token ? token.name : null,
    role: user ? user.role : null
  }
}

module.exports = context
```

The role attribute simply contains the user role.

## Directive

This is the implementation of the `hasRole` directive.

**directives.js**

```js
const { SchemaDirectiveVisitor, ForbiddenError } = require('apollo-server-micro')
const { defaultFieldResolver } = require('graphql')

...

// Custom directive to check if user has role
class hasRole extends SchemaDirectiveVisitor {
  // Field definition directive
  visitFieldDefinition (field) {
    // Get field resolver
    const { resolve = defaultFieldResolver } = field

    // List of roles from directive declaration
    const roles = this.args.roles

    field.resolve = async function (...args) {

      // Get context
      const [, , context] = args

      // Check if user email is in context
      if (roles.indexOf(context.role) === -1) {
        throw new ForbiddenError('You are not authorized for this ressource.')
      }

      // Resolve field
      return resolve.apply(this, args)
    }
  }
}

module.exports = { isAuthenticated: isAuthenticated, hasRole: hasRole }
```

It throws an error if the context role is not included in the directive roles arguments.

## Resolver

By default new users should get the role `USER`.

**resolvers.js**

```js
...
    createUser: async (obj, args, context) => {
      // Check if user already exists
      const user = prepare(await (await usersCollection()).findOne({ email: args.email }))
      if (user) {
        throw new ForbiddenError('User already exists.')
      }

      // Default value
      args.role = args.role ? args.role : 'USER'

      // Hash password
      args.password = await bcrypt.hash(args.password, BCRYPT_ROUNDS)
      args.created = new Date()
      args.created_by = context.name || 'system'
      return prepare((await (await usersCollection()).insertOne(args)).ops[0])
    },
...
```

That's it! You now have role based authorization for your GraphQL resources.

## Login

Before we reach the conclusion of this post series, we wanna ensure that the JWT token expires after week.

**resolvers.js**

```js
...
    loginUser: async (obj, args, context) => {
      // Find user by email and password
      const user = prepare(await (await collection('users')).findOne({ email: args.email }))

      // Compare hash
      if (user && await bcrypt.compare(args.password, user.password)) {
        // Generate and return JWT token
        const token = jwt.sign({ email: user.email, name: (user.firstname + ' ' + user.lastname) }, process.env.JWT_SECRET, { expiresIn: '7d' })
        return { token: token }
      } else {
        // Throw authentication error
        throw new AuthenticationError('Login failed.')
      }
    }
...
```

Adding the `expiresIn: '7d'` option to the sign method ensures that the verify method will throw an error if the token has expired.

## Conclusion

In this post series we covered the basic of authentication with GraphQL and a React App. The solutions provided here are supposed to give you a basic idea and a far from production ready. Having type definitions and access control in the same place is very convenient. That is what like most about GraphQL. Having this flexible layer that unifies access control, api access and schema definitions.

I hope I was able to give you an idea and would be glad if you share your thoughts and question with me 😊.

## Addition

Some final hints on what should be considered when developing an authentication solution like this.

- JWT referesh token > [Auth0 - Refresh Tokens: When to Use Them and How They Interact with JWTs](https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/)
- Token for identity only > [iana - JSON Web Token Claims](https://www.iana.org/assignments/jwt/jwt.xhtml)