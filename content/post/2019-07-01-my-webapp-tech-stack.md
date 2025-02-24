---
title: "My Webapp Tech Stack"
slug: 2019-07-01-my-webapp-tech-stack
date: 2019-07-01T11:38:43+02:00
categories:
 - Web development
tags:
 - tech stack
 - best practices
 - technology
 - methodology
images:
 - /images/toolbox.jpg
---

In this post I am going to present you my personal tech stack for web development. I want to tell more about the technologies and methodologies I am using to build web applications. In that sense it would be glad if you share your stack and methodologies in the comment section.
<!--more-->

## Methodology

My technology-independent methodology and approach to software development:

* Web apps over native apps (aka progressive web apps)
* [The twelfe-factor app](https://12factor.net/)
* [Serverless compute manifesto](https://de.slideshare.net/AmazonWebServices/getting-started-with-aws-lambda-and-the-serverless-cloud/29)

## Technology

For every layer of a modern web application I focus on one technology.

**Language:** JavaScript

JavaScript has become an ubiquitous programming language for web development.

**UI Framework:** [React](https://reactjs.org/)

React is the most popular user interface framework. Whereas Angular dominates the enterprise domain, React is the first choice for consumer apps.

**UI Components:** [Material-UI](https://material-ui.com/)

Material-UI unites Google's design principles and React's versatile components. Designing components and implementing a visual design that adheres to best practices has become a major challenge. Using Material-UI components makes it trivial to build an attractive UI within a short amount of time.

**Client State:** [Apollo Client](https://www.apollographql.com/docs/react/)

Apollo is a [GraphQL](https://graphql.org/) implementation. The Apollo client makes it easy to manage the [local state](https://www.apollographql.com/docs/react/essentials/local-state/) of a React app.

**Backend:** [Apollo Server](https://www.apollographql.com/docs/apollo-server/)

Apollo provides the flexibility and versatility required for designing and implementing complex APIs. The schema language makes it easy to define a solid and coherent data model. Whereas resolvers offer the necessary flexibility to connect the schema to the business and persistence layer.

**Database:** [MongoDB](https://www.mongodb.com/)

The NoSQL approach has become outdated. However, if you design the schema with GraphQL there is no need in applying a schema to the database.

**Platform:** [Zeit Now](https://zeit.co/now)

Zeit's Now offers serverless deployments with little overhead for configuration. It supports monorepos and makes deployments automatically globally available.

**DB Platform:** [mLab](https://mlab.com/)

mLab is the most popular MongoDB hosting platform. I especially appreciate their sandbox plan.

## Minor Tech

Package manager: [yarn](https://yarnpkg.com)  
Auth Token: [JSON Web Token](https://www.apollographql.com/docs/react/essentials/local-state/)  
App Config: [dotenv](https://github.com/motdotla/dotenv)  