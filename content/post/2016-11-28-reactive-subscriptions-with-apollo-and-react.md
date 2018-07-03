---
id: 4137
title: Reactive subscriptions with Apollo and React
date: 2016-11-28T12:27:08+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4137
permalink: /2016/11/28/reactive-subscriptions-with-apollo-and-react/
dsq_thread_id:
  - "5337885808"
dsq_needs_sync:
  - "1"
image: /wp-content/uploads/2016/11/Apollo-Logo.png
categories:
  - Apollo
  - React
tags:
  - apollo
  - publication
  - react
  - reactive
  - reactivity
  - real
  - subscribe
  - subscription
  - time
---
Apollo server and client support real-time subscriptions with web sockets. Compared to Meteor's out of the box real-time communication this is a lot more difficult to set up. With this short tutorial I'll give you an example of how you can get a simple reactive subscriptions into your Apollo/React app. The idea is that you use real-time communication for a specific case only, f.g. sending notifications. We will accomplish this in five steps.

1. Install project requirements.
2. Setup the server schema.
3. Add server resolvers.
4. Setup the client subscription.
5. Modify a component to receive data from a subscription.
<!--more-->
As reference you can have look at this project: [Meteor Apollo Accounts Example](https://github.com/janikvonrotz/meteor-apollo-accounts-example)

The concept of GraphQL subscriptions in Apollo:

![apollo-subscription](https://janikvonrotz.ch/wp-content/uploads/2016/11/Apollo-Subscription.png)

## Install project requirements

I assume you already have an Apollo (server) and React (client) application up and running. To get started we need to install the GraphQL subscription package for the server.

```bash
npm install --save graphql-subscriptions
```

This packages extends the GraphQL schema for subscriptions.

As client server implementation for the subscription communications we use the subscriptions transport web socket package.

```bash
npm install --save subscriptions-transport-ws
```

This package will be used to set up the web socket client and server.

## Setup server schema

Subscriptions are similar to queries and have their own type definition. For our example we add trigger based subscriptions to our schema which means they send data based on specific events.

**server/schema.js**

```
...
const rootSchema = [`

type Post {
  _id: ID
  title: String
}

type Mutation {
  insertPost(
    title: String
  ): Post
  deletePost(
    _id: ID
  ): SuccessResponse
  updatePost(
    _id: ID
    title: String
  ): SuccessResponse
}

type Subscription {
  postInserted: Post
  postDeleted: ID
  postUpdated: Post
}

type Query {
  posts: [Post]
}

schema {
  query: Query
  mutation: Mutation
  subscription: Subscription
}
`]
...
```

Whenever the post type is modified we will submit the mutation in real-time to the clients.

To dispatch data to the web socket connection we have to set up a subscriptions manager.

**server/subscriptions.js**

```js
import { PubSub, SubscriptionManager } from 'graphql-subscriptions';
import { schema } from './index';

const pubsub = new PubSub();
const subscriptionManager = new SubscriptionManager({
  schema,
  pubsub,
});
export { subscriptionManager, pubsub };
```

It is a middle ware between the web socket server and the GraphQL schemas.

Next add a web socket server instance to server application and pass and pass the `SubscrptionManager` config.

**server/main.js**

```js
import { schema, Posts, subscriptionManager } from './index'
import { createApolloServer } from 'meteor/apollo';
import { createServer } from 'http';
import { SubscriptionServer } from 'subscriptions-transport-ws';

const WS_PORT = process.env.WS_PORT || 8080;

createApolloServer({
  schema: schema,
});

const httpServer = createServer((request, response) => {
  response.writeHead(404);
  response.end();
});
httpServer.listen(WS_PORT, () => console.log(
  `Websocket Server is now running on http://localhost:${WS_PORT}`
));
const server = new SubscriptionServer({ subscriptionManager }, httpServer);
```

## Add server resolvers

First add the resolvers for the subscription queries.

**server/resolvers.js**

```js
...
const resolvers = {
  Query: { ... },
  Mutation: { ... },
  Subscription: {
    postInserted(post) {
      return post;
    },
    postDeleted(_id) {
      return _id;
    },
    postUpdated(post) {
      return post;
    },
  },
}
...
```

Then tell the other mutation resolvers to dispatch mutation data. I'm using Meteor and MongoDB to store the post data. It is not required that you have the same setup. Just make sure that a mutation event triggers a publication on the `pubsub` manager.

**sever/resolver.js**

```js
import { Posts, pubsub } from './index'
...
Mutation: {
    insertPost(root, args, context){
        return Posts.insert(args, (error, response) => {
          if(response){
            args._id = response
            pubsub.publish('postInserted', args)
          }
        })
    },
    deletePost(root, args, context){
        return { success: Posts.remove(args, (error, response) => {
          if(response){
            pubsub.publish('postDeleted', args._id)
          }
        }) }
    },
    updatePost(root, args, context){
        let _id = args._id
        delete args._id
        return { success: Posts.upsert(_id, { $set: args }, (error, response) => {
          if(response){
            args._id = _id
            pubsub.publish('postUpdated', args)
          }
        }) }
    },
...
```

Our server is now ready to send subscriptions data in real-time.

## Setup the client subscription

Now we configure our client to receive data sent by the publication. For this we configure a Apollo network interface.

**client/Subscriptions.js**

```js
import { print } from 'graphql-tag/printer';

const addGraphQLSubscriptions = (networkInterface, wsClient) => Object.assign(networkInterface, {
  subscribe: (request, handler) => wsClient.subscribe({
    query: print(request.query),
    variables: request.variables,
  }, handler),
  unsubscribe: (id) => {
    wsClient.unsubscribe(id);
  },
});

export default addGraphQLSubscriptions;
```

And connect it to our Apollo client.

**client/ApolloClient.js**

```js
import ApolloClient, { createNetworkInterface } from 'apollo-client'
import { Client } from 'subscriptions-transport-ws';
import { addGraphQLSubscriptions } from './index';

const wsClient = new Client('ws://localhost:8080');

const networkInterface = createNetworkInterface({ uri: '/graphql' })

const networkInterfaceWithSubscriptions = addGraphQLSubscriptions(
  networkInterface,
  wsClient,
);

const client = new ApolloClient({
  networkInterface: networkInterfaceWithSubscriptions,
})

export default client
```

There is nothing more to do. We can now subscribe to a publication and receive data from the server.

## Modify a component to receive data from a subscription

Finally I give an example of a modified React component that is able to mutate a post model list, make subscriptions and receive emitted events from a publication. 

![postlist-react-component-apollo-subscription](https://janikvonrotz.ch/wp-content/uploads/2016/11/PostList-React-component-Apollo-subscription.gif)

**client/PostList.js**

```js
import React from 'react'
import { graphql } from 'react-apollo'
import gql from 'graphql-tag'
import { ApolloClient, Notification } from './index'
import _ from 'underscore'

const postInserted = gql`
  subscription postInserted {
    postInserted {
      _id
      title
    }
  }
`;

const postDeleted = gql`
  subscription postDeleted {
    postDeleted
  }
`;

const postUpdated = gql`
  subscription postUpdated {
    postUpdated {
      _id
      title
    }
  }
`;

class PostList extends React.Component {

  constructor(props) {
    super(props)
    this.state = {}
    this.subscription = null
  }

  componentWillReceiveProps(nextProps) {
    if (!this.subscription &amp;&amp; !nextProps.data.loading) {
      let { subscribeToMore } = this.props.data
      this.subscription = [subscribeToMore(
        {
          document: postInserted,
          updateQuery: (previousResult, { subscriptionData }) => {
            previousResult.posts.push(subscriptionData.data.postInserted)
            return previousResult
          },
        }
      ),
      subscribeToMore(
        {
          document: postDeleted,
          updateQuery: (previousResult, { subscriptionData }) => {
            previousResult.posts = _.without(previousResult.posts, _.findWhere(previousResult.posts, {
              _id: subscriptionData.data.postDeleted
            }));
            return previousResult
          },
        }
      ),
      subscribeToMore(
        {
          document: postUpdated,
          updateQuery: (previousResult, { subscriptionData }) => {
            previousResult.posts = previousResult.posts.map((post) => {
              if(post._id === subscriptionData.data.postUpdated._id) {
                return subscriptionData.data.postUpdated
              } else {
                return post
              }
            })
            return previousResult
          },
        }
      )]
    }
  }

  async insert(event) {
    event.preventDefault()

    let { insertPost, data } = this.props
    let { insertTitle } = this.refs
    let title = insertTitle.value

    try {
      const response = await insertPost({ title })
      Notification.success(response)
    } catch (error) {
      Notification.error(error)
    }
  }

  async delete(_id, event) {
    event.preventDefault()

    let { deletePost, data } = this.props

    try {
      const response = await deletePost({ _id })
      Notification.success(response)
    } catch (error) {
      Notification.error(error)
    }
  }

  edit(_id) {
    this.setState({edit: _id})
  }

  async update(_id) {
    let { updatePost, data } = this.props

    let { updateTitle } = this.refs
    let title = updateTitle.value

    try {
      const response = await updatePost({ _id, title })
      Notification.success(response)
      this.setState({edit: null})
    } catch (error) {
      Notification.error(error)
    }
  }

  render() {
    let { posts, loading } = this.props.data
    let { edit } = this.state

    return loading ? (<p>Loading...</p>) : (
      <div>
        <h2>Posts</h2>
        <form onSubmit={this.insert.bind(this)}>
          <label>Title: </label>
          <input
          defaultValue="Untitled"
          type="text"
          required="true"
          ref="insertTitle" />
          <br />
          <button type="submit">Insert Post</button>
        </form>
        { posts ? (
          <ul>
            {posts.map((post) => {
              return (
                edit === post._id ?
                  <li key={post._id}>
                    <input
                    ref="updateTitle"
                    defaultValue={post.title}
                    size="50"
                    type="text" />
                    <button onClick={this.update.bind(this, post._id)}>Update</button>
                  </li> : <li key={post._id}>
                    <span onClick={this.edit.bind(this, post._id)}>{post.title} </span>
                    <button onClick={this.delete.bind(this, post._id)}>Delete</button>
                  </li>
              )
            })}
          </ul>
        ) : <p>No posts available.</p> }
      </div>
    )
  }
}

const query = gql`
query getData {
  posts {
    _id
    title
  }
}
`

const insertPost = gql`
mutation insertPost($title: String) {
  insertPost(title: $title){
    _id
  }
}
`

const deletePost = gql`
mutation deletePost($_id: ID) {
  deletePost(_id: $_id){
    success
  }
}
`

const updatePost = gql`
mutation updatePost($_id: ID, $title: String) {
  updatePost(_id: $_id, title: $title){
    success
  }
}
`

PostList = graphql(updatePost, {
  props({ mutate }) {
    return {
      updatePost({_id, title}) {
        return mutate({ variables: { _id, title }})
      }
    }
  },
})(graphql(deletePost, {
  props({ mutate }) {
    return {
      deletePost({_id}) {
        return mutate({ variables: { _id }})
      }
    }
  },
})(graphql(insertPost, {
  props({ mutate }) {
    return {
      insertPost({title}) {
        return mutate({ variables: { title }})
      }
    }
  },
})(graphql(query)(PostList))))

export default PostList
```

Hope you liked this tutorial.

## Source

[Apollo Example GitHunt server](https://github.com/apollostack/GitHunt-API)

[Apollo Example GitHunt React client](https://github.com/apollostack/GitHunt-React)

[GraphQL Subscriptions in Apollo Client](https://dev-blog.apollodata.com/graphql-subscriptions-in-apollo-client-9a2457f015fb)

[A proposal for GraphQL subscriptions](https://dev-blog.apollodata.com/a-proposal-for-graphql-subscriptions-1d89b1934c18#.)

[GraphQL Subscriptions in Apollo Client](https://dev-blog.apollodata.com/graphql-subscriptions-in-apollo-client-9a2457f015fb)