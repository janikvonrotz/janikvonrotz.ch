---
title: "Apollo GraphQl server and client sorting"
slug: apollo-graphql-server-and-client-sorting
date: 2020-01-29T14:05:23+01:00
categories:
 - JavaScript development
tags:
 - apollo
 - graphql
 - sorting
 - javascript
images:
 - /images/metal network.jpg
---

GraphQl is not opinionated about sorting and pagination. It is up to you to implement the sorting for your query. I've seen various approaches doing that, but none seemed elegant. After compiling a few blog posts and tutorials I came up with the following solution.
<!--more-->

We want to make a selected query sortable. Therefore we add an optional parameter to the query in the schema. The parameter is a custom input type. It consists of a field name and a sort order. The field property is a string and the order property is an enum. Order is either ascending or descending.

Here is my schema example:

```gql
enum Order {
  ASC
  DESC
}

input SortBy {
  field: String!
  order: Order!
}

type Group {
  id: String!
  name: String!
}

type Tenant {
  id: String!
  name: String!
  group: Group
}

type Query {
  tenants(sortBy: SortBy): [Tenant]
}
```

The query in action would look like this:

```gql
{
  tenants(sortBy: { field: "name", order: DESC }) {
    id
    name
  }
}
```

Implementing the resolver is up to you.

Here is an example of my MongoDB collection lookup.

```js
// ...
const resolvers = {
  Query: {
    tenants: async (obj, args, context) => {
      const sortBy = {}
      if (args.sortBy) {
        sortBy[args.sortBy.field] = args.sortBy.order === 'ASC' ? 1 : -1
      }
      return (await (await collection('tenants')).find({}).sort(sortBy).toArray()).map(prepare)
    },
    // ...
```

Quite simple isn't it? But, what if we wanna sort by a nested property. Let's say `tenant.group.name`.

It simply is not possible to accomplish this in GraphQl. Subtypes are resolved in an isolated resolver.

```js
// ...
const resolvers = {
  Query: {
      // ...
    },
    // ...
  },
  Tenant: {
    group: async (obj, args, context) => {
      return groupResolver.Query.group(obj, { id: obj.group }, context)
    }
  }
  // ...
```

Now instead of sorting on the server side you could to that on the client side.

The main difference is that you do not have access to the full data set. Assuming you only return a subset of the data from the Graphql api, you only sort a part of the full data on the client.

Here is a sort function that provides a similar functionality as we've seen above.

```js
const compareValues = (key, order = 'ASC') => {
  const keys = key.split('.')

  return (a, b) => {
    let i = 0

    // Access nested property
    while (i < keys.length) {
      a = a[keys[i]]
      b = b[keys[i]]
      i++
    }

    const varA = (typeof a === 'string') ? a.toUpperCase() : a
    const varB = (typeof b === 'string') ? b.toUpperCase() : b

    let comparison = 0
    if (varA > varB) {
      comparison = 1
    } else if (varA < varB) {
      comparison = -1
    }
    return (
      (order === 'DESC') ? (comparison * -1) : comparison
    )
  }
}
```

This function can be passed to the array sort method.

```js
const tenants = [
  {
    id: 1,
    name: 'A-Tenant',
    group: {
      id: 1,
      name: 'A-Group'
    }
  },
  {
    id: 2,
    name: 'B-Tenant',
    group: {
      id: 2,
      name: 'B-Group'
    }
  },
  {
    id: 3,
    name: 'C-Tenant',
    group: {
      id: 3,
      name: 'C-Group'
    }
  }
]

tenants.sort(compareValues('group.name', 'DESC'))
```

I would be really interested if you have another solution for the nested sorting problem.
