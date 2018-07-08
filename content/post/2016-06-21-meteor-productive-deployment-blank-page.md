---
title: Meteor productive deployment blank page
date: 2016-06-21T10:16:07+00:00
author: Janik von Rotz
permalink: /2016/06/21/meteor-productive-deployment-blank-page/
dsq_thread_id:
  - "4927016387"
image: /wp-content/uploads/2015/12/Meteor-Logo.png
categories:
  - Blog
  - Meteor
tags:
  - blank
  - deployment
  - empty
  - es6
  - git
  - heroku
  - jsx
  - master
  - meteor
  - minifier
  - minify
  - package
  - page
  - problem
  - production
  - push
  - react
  - run
---
First some background. I've built a Meteor app and decided to deploy it to Heroku. There are many short and simple guides out there on how to do that. As I did so and opened the App for the first on Heroku I simply received a blank page. Where did my javascript code go? Heroku logs didn't tell much.
<!--more-->
After googling this issue I was told that Heroku launches the App in productive mode or at least the buildpack I've used does so. So I tested this locally with `meteor --production` and surprise got a blank page as well. So what does this flag change in the build process? It simply minifies the javascript code. So that is the problem. As my app is built with React written in ES6 the minification probably doesn't work. The solution is quite simple, remove the Meteor package `standard-minifier-js` from the project.