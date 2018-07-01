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

<img src="https://janikvonrotz.ch/wp-content/uploads/2016/11/Apollo-Subscription.png" alt="apollo-subscription" width="800" height="653" class="aligncenter size-full wp-image-4146" />

## Install project requirements

I assume you already have an Apollo (server) and React (client) application up and running. To get started we need to install the GraphQL subscription package for the server.

[code language="shell"]
npm install --save graphql-subscriptions
[/code]

This packages extends the GraphQL schema for subscriptions.

As client server implementation for the subscription communications we use the subscriptions transport web socket package.

[code language="shell"]
npm install --save subscriptions-transport-ws
[/code]

This package will be used to set up the web socket client and server.

## Setup server schema

Subscriptions are similar to queries and have their own type definition. For our example we add trigger based subscriptions to our schema which means they send data based on specific events.

**server/schema.js**

[code]
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
[/code]

Whenever the post type is modified we will submit the mutation in real-time to the clients.

To dispatch data to the web socket connection we have to set up a subscriptions manager.

**server/subscriptions.js**

[code language="javascript"]
import { PubSub, SubscriptionManager } from 'graphql-subscriptions';
import { schema } from './index';

const pubsub = new PubSub();
const subscriptionManager = new SubscriptionManager({
  schema,
  pubsub,
});
export { subscriptionManager, pubsub };
[/code]

It is a middle ware between the web socket server and the GraphQL schemas.

Next add a web socket server instance to server application and pass and pass the `SubscrptionManager` config.

**server/main.js**

[code language="javascript"]
import { schema, Posts, subscriptionManager } from './index'
import { createApolloServer } from 'meteor/apollo';
import { createServer } from 'http';
import { SubscriptionServer } from 'subscriptions-transport-ws';

const WS_PORT = process.env.WS_PORT || 8080;

createApolloServer({
  schema: schema,
});

const httpServer = createServer((request, response) =&gt; {
  response.writeHead(404);
  response.end();
});
httpServer.listen(WS_PORT, () =&gt; console.log(
  `Websocket Server is now running on http://localhost:${WS_PORT}`
));
const server = new SubscriptionServer({ subscriptionManager }, httpServer);
[/code]

## Add server resolvers

First add the resolvers for the subscription queries.

**server/resolvers.js**

[code language="javascript"]
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
[/code]

Then tell the other mutation resolvers to dispatch mutation data. I'm using Meteor and MongoDB to store the post data. It is not required that you have the same setup. Just make sure that a mutation event triggers a publication on the `pubsub` manager.

**sever/resolver.js**

[code language="javascript"]
import { Posts, pubsub } from './index'
...
Mutation: {
    insertPost(root, args, context){
        return Posts.insert(args, (error, response) =&gt; {
          if(response){
            args._id = response
            pubsub.publish('postInserted', args)
          }
        })
    },
    deletePost(root, args, context){
        return { success: Posts.remove(args, (error, response) =&gt; {
          if(response){
            pubsub.publish('postDeleted', args._id)
          }
        }) }
    },
    updatePost(root, args, context){
        let _id = args._id
        delete args._id
        return { success: Posts.upsert(_id, { $set: args }, (error, response) =&gt; {
          if(response){
            args._id = _id
            pubsub.publish('postUpdated', args)
          }
        }) }
    },
...
[/code]

Our server is now ready to send subscriptions data in real-time.

## Setup the client subscription

Now we configure our client to receive data sent by the publication. For this we configure a Apollo network interface.

**client/Subscriptions.js**

[code language="javascript"]
import { print } from 'graphql-tag/printer';

const addGraphQLSubscriptions = (networkInterface, wsClient) =&gt; Object.assign(networkInterface, {
  subscribe: (request, handler) =&gt; wsClient.subscribe({
    query: print(request.query),
    variables: request.variables,
  }, handler),
  unsubscribe: (id) =&gt; {
    wsClient.unsubscribe(id);
  },
});

export default addGraphQLSubscriptions;
[/code]

And connect it to our Apollo client.

**client/ApolloClient.js**

[code language="javascript"]
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
[/code]

There is nothing more to do. We can now subscribe to a publication and receive data from the server.

## Modify a component to receive data from a subscription

Finally I give an example of a modified React component that is able to mutate a post model list, make subscriptions and receive emitted events from a publication. 

<img src="https://janikvonrotz.ch/wp-content/uploads/2016/11/PostList-React-component-Apollo-subscription.gif" alt="postlist-react-component-apollo-subscription" width="1165" height="330" class="aligncenter size-full wp-image-4154" />

**client/PostList.js**

[code language="javascript"]
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
          updateQuery: (previousResult, { subscriptionData }) =&gt; {
            previousResult.posts.push(subscriptionData.data.postInserted)
            return previousResult
          },
        }
      ),
      subscribeToMore(
        {
          document: postDeleted,
          updateQuery: (previousResult, { subscriptionData }) =&gt; {
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
          updateQuery: (previousResult, { subscriptionData }) =&gt; {
            previousResult.posts = previousResult.posts.map((post) =&gt; {
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

    return loading ? (&lt;p&gt;Loading...&lt;/p&gt;) : (
      &lt;div&gt;
        &lt;h2&gt;Posts&lt;/h2&gt;
        &lt;form onSubmit={this.insert.bind(this)}&gt;
          &lt;label&gt;Title: &lt;/label&gt;
          &lt;input
          defaultValue=&quot;Untitled&quot;
          type=&quot;text&quot;
          required=&quot;true&quot;
          ref=&quot;insertTitle&quot; /&gt;
          &lt;br /&gt;
          &lt;button type=&quot;submit&quot;&gt;Insert Post&lt;/button&gt;
        &lt;/form&gt;
        { posts ? (
          &lt;ul&gt;
            {posts.map((post) =&gt; {
              return (
                edit === post._id ?
                  &lt;li key={post._id}&gt;
                    &lt;input
                    ref=&quot;updateTitle&quot;
                    defaultValue={post.title}
                    size=&quot;50&quot;
                    type=&quot;text&quot; /&gt;
                    &lt;button onClick={this.update.bind(this, post._id)}&gt;Update&lt;/button&gt;
                  &lt;/li&gt; : &lt;li key={post._id}&gt;
                    &lt;span onClick={this.edit.bind(this, post._id)}&gt;{post.title} &lt;/span&gt;
                    &lt;button onClick={this.delete.bind(this, post._id)}&gt;Delete&lt;/button&gt;
                  &lt;/li&gt;
              )
            })}
          &lt;/ul&gt;
        ) : &lt;p&gt;No posts available.&lt;/p&gt; }
      &lt;/div&gt;
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
[/code]

Hope you liked this tutorial.

## Source

[Apollo Example GitHunt server](https://github.com/apollostack/GitHunt-API)

[Apollo Example GitHunt React client](https://github.com/apollostack/GitHunt-React)

[GraphQL Subscriptions in Apollo Client](https://dev-blog.apollodata.com/graphql-subscriptions-in-apollo-client-9a2457f015fb)

[A proposal for GraphQL subscriptions](https://dev-blog.apollodata.com/a-proposal-for-graphql-subscriptions-1d89b1934c18#.)

[GraphQL Subscriptions in Apollo Client](https://dev-blog.apollodata.com/graphql-subscriptions-in-apollo-client-9a2457f015fb)