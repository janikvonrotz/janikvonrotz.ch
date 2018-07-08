---
title: Graphql with Apollo, Meteor and React
date: 2016-10-09T19:39:30+00:00
author: Janik von Rotz
permalink: /2016/10/09/graphql-with-apollo-meteor-and-react/
dsq_thread_id:
  - "5209861289"
image: /wp-content/uploads/2016/10/react-meteor-graphl-apollo-1200x356.jpg
categories:
  - Apollo
  - Blog
  - Meteor
  - React
tags:
  - apollo
  - build
  - graphql
  - javascript
  - meteor
  - project
  - react
  - stack
  - system
---
For my last project I had to build a web application to administrate a MongoDB database. Due to using Meteor quite a lot I heard about Graphql and the Apollostack. Graphql, which is a specification done by Facebook engineers, promises to be the better REST API (which I hope it is). I became curious and decided the build the server API with Apollo. First I tried to evade using the Meteor as build system as I don't want to get too accustomed to this full-stack ecosystem. However, building a live-reload server and client build system in ES6 with Node.js, Babel and Webpack was simply too much work compared to building this simple web app. So in result this was my stack:
<!--more-->

- Build system > Meteor
- Graphql client > React and Apollo
- Graphql server > Express and Apollo

Learning Graphql and how to use Apollo in this environment was a bit of a challenge. As Apollo is still the new kid on the block and its documentation lacks a few chapters I had to checkout the source code a few other projects to learn more about it. Graphl involves a few new concepts and makes you build APIs differently. Whereas in REST you build a hierarchical, untyped and strict interface in Graphql the possibilities first seem endless. Apollo delivers a high grade of flexibility in designing schemas and abstraction of data gathering with a set of resolvers. This might by overwhelming at the beginning, but is a lot of fun once you understand it. Despite the project was quite easy I had to solve a lot of issues. To make life easier for fellow Meteor, React, Apollo developer I'll share the source code of my project and list the most interesting problems I've solved:

GitHub repository: [https://github.com/janikvonrotz/Apometact](https://github.com/janikvonrotz/Apometact)

## Custom scalar data types

In all of my models I'm using the default Date data type. As Apollo basically only supports strings and numbers to be transported I came up with this solution.

**schema.js**

```
scalar Date
...
type Category {
  id: String
  label: String
  createdAt: Date
}
...
```

Define a new scalar, which is basically another type.

**resolvers.js**

```
  Date: {
    __parseValue(value) {
      return new Date(value); // value from the client
    },
    __serialize(value) {
      return value.toISOString(); // value sent to the client
    },
    __parseLiteral(ast) {
      return ast.value;
    },
  },
```

And tell Apollo how to parse it when receiving queries or dispatch data.

Now you can send date types as string to the Graphql interface.

## Multiple queries and mutators in a React component

All the examples as part of the source code of projects or the documentation didn't tell how to use multiple mutators and queries in a React component. Here's how I've solved it: 

**CategoryEdit.js**

```
...
const updateMutation = gql`
mutation updateCategory($id: String, $label: String) {
  updateCategory(id: $id, label: $label)
}
`;
const deleteMutation = gql`
mutation deleteCategory($id: String) {
  deleteCategory(id: $id)
}
`;
const CategoryEditWithMutation = graphql(updateMutation, {
  props({ mutate }) {
    return {
      updateCategory({id, label}) {
        return mutate({ variables: { id, label }});
      }
    };
  },
})(graphql(deleteMutation, {
  props({ mutate }) {
    return {
      deleteCategory(id){
        return mutate({ variables: { id }});
      }
    };
  },
})(CategoryEdit));

const query = gql`
query getCategory($id: String) {
  category(id: $id) {
    id
    label
  }
}
`;
const CategoryEditWithData = graphql(query, {
  options: (ownProps) => ({ variables: { id: ownProps.params.id } }),
})(CategoryEditWithMutation);

export default CategoryEditWithData;
...
```

Multiple queries can be lined up in one Graphql tag whereas mutators have to be divided (as long you don't wish to run multiple mutators in one call).

## Mongoose promises in Apollo resolver

Node.js runs asynchronously and so does Apollo. Instead of retrieving data from another endpoint you'll pass a callback function which will be executed once the endpoint delivered its data. When defining resolvers in Apollo you have to switch roles. Instead of returning pure data in a resolve function you have to return a promise, which can be executed in a callback.

**resolvers.js**   

```
...
categories(root, args){
  return Categories.find({}).then((items) => {
    return items.map((item) => {
      item.id = item._id;
      return item;
    });
  });
},
...
```

Define with `then` and `catch` the returned promise.

**mongoose.js**

```
...
Mongoose.Promise = global.Promise;
...
var CategoriesSchema = new Schema({
  label: String,
  createdAt: Date
});
var Categories = mongo.model('Categories', CategoriesSchema);
...
```

Set the ES6 default Promise object for Mongoose.

## Load configurations the right way

This is not necessarily a use case for this kind of project, but I would like to share it anyway. Loading configurations works kind of different in Meteor than it does in Node.js.

First an example of how I load configurations in Node.js.

**node-config.js**

```
import fs from 'fs';

var config = {}
var path = `./${process.env.NODE_ENV || 'development'}.json`
try {
  fs.accessSync(path, fs.F_OK);
  console.log(`Load configurations from ${path}`)
  config = require(path);
} catch (error) {
  console.log(error)
}

// load environment variables
config = Object.assign(config, process.env);

export default config;
```

Based on the environment variable a json file will be chosen and loaded. All the configurations then will be overwritten by the available environment variables (necessary when deploying to most app host services like Heroku or Modulus).

**meteor-config.js**

```
import { Meteor } from 'meteor/meteor';

var config = Object.assign(Meteor.settings.private, Meteor.settings.public);

export default config
```

The meteor app can be started with this npm command: `"start": "meteor --production --settings ./production.json",`

Now in both cases the code creates an object containing configurations.
