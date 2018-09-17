---
title: Authenticate Meteor accounts with the Apollo GraphQL API
date: 2016-11-12T10:21:20+00:00
author: Janik von Rotz
slug: authenticate-meteor-accounts-with-the-apollo-graphql-api
dsq_thread_id:
  - "5298095930"
images:
  - /wp-content/uploads/2016/11/Apollo-Logo.png
categories:
  - Apollo
tags:
  - accounts
  - apollo
  - authentication
  - graphql
  - login
  - logout
  - meteor
  - module
  - package
  - user
---
One of the popular features of Meteor is its accounts package. As you know, it makes it fairly easy to add a user authentication solution to your Meteor app or add support for different oAuth services. With the possibility to integrate an Apollo GraphQL API into your Meteor app this became a bit more difficult. The Apollo stack does not support an out of the box solutions to authenticate users with Meteor accounts. Jonas Helfer, one of the Apollo core devs, proposed [two ways to authenticate users with your app](https://dev-blog.apollodata.com/a-guide-to-authentication-in-graphql-e002a4039d1):
<!--more-->

1. Let the web server (e.g. express or nginx) take care of authentication.
2. Handle authentication in GraphQL itself.

Of course this was not meant especially for Meteor, however, it gave some direction where this is going for Meteor. Obviously the first solution is not very attractive, if possible you want to use one API to exchange data only. So I looked for a solution where it was possible to extend an existing GraphQL schema and authenticate Meteor user accounts without the hassle to write GraphQL resolvers or client methods. Nicolas Lopes built exactly what I was looking for:

**[Meteor Apollo Accounts](https://github.com/nicolaslopezj/meteor-apollo-accounts)**
An implementation of Meteor Accounts only in GraphQL with Apollo.

The package was still early in development, so I decided to help him out. Not only because I wanted this package to work, but also because I never contributed to an open source project before. Nicolas looked for someone building him an [example app](https://github.com/nicolaslopezj/meteor-apollo-accounts/issues/3). Perfect for me. One month later I've built an example app that implements all the Meteor accounts features one would expect:

**[Meteor Apollo Accounts Example](https://github.com/janikvonrotz/meteor-apollo-accounts-example)**
Example app implementing Meteor Apollo Accounts.

Together we solved quite a lot of issues and learned a lot. Building the example app was kind of software testing for Nicolas and him telling me about his style of coding and PR requirements taught me a lot about open source contribution and coding skills. So in conclusion this was a great experience.

