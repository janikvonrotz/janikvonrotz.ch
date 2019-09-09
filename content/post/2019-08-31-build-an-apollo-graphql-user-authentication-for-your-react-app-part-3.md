---
title: "2019 08 31 Build an Apollo Graphql User Authentication for Your React App Part 3"
slug: 2019-08-31-build-an-apollo-graphql-user-authentication-for-your-react-app-part-3
date: 2019-09-01T10:31:27+02:00
categories:
 - Blog
tags:
 - hello
 - world
images:
 - /images/logo.png
draft: true
---

api/schema.js

```js
const { gql } = require('apollo-server-micro')

// GraphQL schema
const typeDefs = gql`

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
  users: [User] @hasRole(roles: [User])
}

type Mutation {
  createUser(email: String!, password: String!, firstname: String!, lastname: String!, role: Role): User
  updateUser(id: String!, email: String, password: String, firstname: String, lastname: String, role: Role): Response
  deleteUser(id: String!): Response
}
`
module.exports = typeDefs
```

api/context.js

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
    role: user ? user.role : 'USER'
  }
}

module.exports = context
```

api/directives.js

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

api/resolvers.js

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