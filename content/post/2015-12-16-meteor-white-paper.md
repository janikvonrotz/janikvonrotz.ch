---
title: Meteor White Paper
date: 2015-12-16T21:05:20+00:00
author: Janik von Rotz
slug: meteor-white-paper
dsq_thread_id:
  - "4409826194"
image: /wp-content/uploads/2015/12/Meteor-Logo.png
categories:
  - Meteor
tags:
  - about
  - javascript
  - meteor
  - paper
  - technology
  - white
---
# Introduction

Today's internet would not work without web applications. Web applications made the web dynamic and extended its targeted audience. The applications made it possible for companies to create new business models and sell their goods and services online. They became popular due to the ubiquity of web browsers, and the convenience of using a web browser as a client (Wikipedia, 2015). Undoubtedly, the technology to build web applications is the cornerstone to enabling the enterprises to keep up with the pace of business. Life cycles of products have become shorter while the demand raised at the same time. Enterprises have to accelerate their application development process to stay competitive in the future.
<!--more-->
However, there is a fundamental problem in how web applications worked so far. The web was originally designed to work in the same way that mainframes worked in the 70s. The application server rendered a screen and sent it over the network to a dumb terminal. Whenever the user did anything, that server had to render a whole new screen. This model served the web well for more than a decade  (Meteor Development Group, 2015).

Nowadays, users want to see their data on the screen as fast as possible and without interruption. Performance matters more than ever. Only native apps were able to satisfy the users this far. The web applications had to become more reactive and the data feed real-time. To solve this problem, developers have invented things such as Ajax which allows a browser to call data from the server and update the site passively. Others have tried to improve the amount of requests a server can handle and reduce the time it takes to send response. Nonetheless, most of these innovations were workarounds which did not solve the actual problem at hand. Until now.

Even more genius developers, namely the Meteor Development Group, have created Meteor. Meteor is a young open-source JavaScript web application framework written in Node.js. Instead of the common slow request-reply pattern Meteor uses the modern publish-subscribe strategy to automatically propagate data changes to the clients (Krill, 2015). Meteor will change the way interactions with web applications are done. Funded with $11.2 M by Y Combinator in July 2012 the Meteor startup has grown up and now has a crucial impact on the world of web development.

# All about Meteor

**Framework**

As mentioned Meteor is a JavaScript web framework, this simply means that the server side code is also written in JavaScript. A web application framework (WAF) is a collection of software that is designed to support the development of web based services. They aim to alleviate the overhead associated with the development of web projects.

**Full stack JavaScript**

Until now developers had to use different coding languages such as PHP or Ruby and used JavaScript on the client only. Now with Meteor developers are able to write full-stack JavaScript apps. This means JavaScript code that runs on the client and server. Behind the scenes of Meteor is Node.js. Node.js is an open-source, cross-platform runtime environment, which means that you can run JavaScript server code on any operating system.

**Real-time**

With Meteor there's no need to write synchronization code. Data changes on the server are automatically transmitted to the clients. This mechanism is very essential to this topic and makes Meteor a real game changer.

**Project**

The Meteor project is collaboration of many developers. Everybody that doesn't lack the knowledge can help improve Meteor on GitHub. GitHub is the most popular code collaboration platform on the internet.

**Persistence Layer**

Meteor uses the Mongo database software to store data persistently. Mongo DB is a non-relational document oriented database system. Compared to the traditional table based relational database system, data is stored in a dynamic schema.

**Architecture**

The Architecture is built on the latest advancements in web technologies. Powerful components that communicate with minor overhead and work well together are essential to the Meteor architecture.

![Meteor architecture](/wp-content/uploads/2015/12/Meteor-architecture.png)

Meteor invented a new, dynamic architecture for building modern web and mobile applications. The framework provides the underpinning for modern programming components, such as a reactive user interface (UI), Web Socket-based Data transport, APIs that work identically on the server client, and a unified build system.

Yet Meteor still is a young player compared to other frameworks on the market. Blaze the UI component that comes with Meteor still lacks functionality. Namely the two-way data binding is a major challenge for scaling applications. However, Meteor integrates well with other UI frameworks such as Google's Angular or Facebook's React (McKay, 2015).

The articles referred in this paper are non-scientific and all of them are online sources. Due to Meteor still being a new technology there doesn't exist any scientific literature.

# Key features of Meteor

Why should a development company invest their important resources in this young technology? Considering the fast pace of web technology, it is important to make thoughtful decisions and not blindly invest in every hype that occurs. Here is just an abstract of the key factors which speak for Meteor.

**Popularity**

There are more JavaScript developers out there than ever before. Its popularity makes it easy to find willing developers that will learning to work with Meteor (Zapponi, 2015). Tens of thousands of developers use Meteor to build new applications every day and Meteor is now the top 10 starred projects on GitHub (GitHub, 2015). As a company it's important to maintain attractivity for new talents. This is an often overlooked aspect when making organizational technology decisions.

**Simplicity**

Meteor aims to make it possible to build a prototype in a day or two and a ready production app in a few week (Meteor, 2015). Developers can easy manage projects dependencies, do code factoring and run the deployment of the app with Meteors command line tool. No more hassle with third party tools.

**Promising**

The market research institute Gartner describes Meteor as the easiest, fastest and most cost effective platform for application development that has existed in many years (Ratkevic, 2015). This makes Meteor a promising technology for the future. Meteor is also very well-funded, and their Galaxy cloud app hosting platform is revenue-generating. The Meteor Development Group received $31.2M in funding so far ( Dascalescu, 2015).

**Ubiquitous**

Thanks to Node.js Meteor runs everywhere. With the Internet of Things (IoT) more and more devices get connected, are exchanging data and run code. This huge growth in connected systems will be the next big challenge. Data growth will be essentially limitless and scalability will present a whole new hurdle. Only frameworks such as Node.js that are built for this kind of environment will be able to face these challenges (Robinson , 2015).

**Performance**

Meteor is based on Node.js. This means Meteor inherits all the advantages of Node.js. The most often touted feature of Node is performance. It's light overhead and event driven architecture makes it possible to serve an application to even more people. PayPal used Node, doubled the number of request-per-second and reduced response time by 35% compared to a parallel application written in Java (Harrell, 2015).

**Community**

Communities are very important to an open-source project. Having skilled people that are interested in improving a project is an essential keyfor success. Luckily, Meteor's community is already huge and vibrant. There's a ton of helpful resources that have spawned from people's love for the framework.

**Real-time**

One of the killer features of Meteor is its reactivity. It uses the Distributed Data Protocol (DDP) to fetch structured data from a server and receive live updates when that data changes. DDP which was originally developed by the Meteor project is now a standard set of names for messages that most web-socket-using application have implemented (Meteor, 2015). Application that are built with Meteor are real-time by default. It's inevitable that that users will expect to work near-instantaneously. With Meteor nobody has to worry about this anymore.

# Further recommendations

What else is there to tell about Meteor. It's a wonderful piece of technology that overwhelms with innovative features and not yet seen simplicity. Real-time communication with higher performance that's like a dream came true. Write a web application in one programming language only and distribute the app to every device will make every developer happier. Well-known companies are already using Meteor in their productive environment and serve thousands of users a day. It is a proven and reliable framework with a lot of potential. Why not use Meteor in the next chat or IoT application and make it real-time by default? Struggling with learning the tech skills needed? There are is a huge and vibrant community that will take care of that. Meteor has an answer to almost every question. No more hassle with third party libraries in your project. With Meteor developers get an all in one box to create their next big project. And if your dev team has not heard about Meteor yet make sure to tell them.

# Literature

Dascalescu, D. (2015, 12 15). _Why Meteor?_ Retrieved from Dan Dascalescu's Wiki: https://wiki.dandascalescu.com/essays/why\_meteor

GitHub. (2015, 12 15). _Top Starred Projects_. Retrieved from GitHub: https://github.com/stars?direction=desc&sort=stars

Harrell, J. (2015, 12 15). _Node.js at PayPal_. Retrieved from PayPal Blog: https://www.paypal-engineering.com/2013/11/22/node-js-at-paypal/

Krill, P. (2015, 12 15). _Meteor aims to make JavaScript programming fun again_. Retrieved from Java World: http://www.javaworld.com/article/2080842/java-app-dev/meteor-aims-to-make-javascript-programming-fun-again.html

McKay, S. (2015, 12 15). _Comparing Performance of Blaze, React, Angular-Meteor and Angular 2 with Meteor_. Retrieved from Meteor Blog: http://info.meteor.com/blog/comparing-performance-of-blaze-react-angular-meteor-and-angular-2-with-meteor

Meteor. (2015, 12 15). _DDP_. Retrieved from Meteor: https://www.meteor.com/ddp

Meteor. (2015, 12 15). _Features_. Retrieved from Meteor: https://www.meteor.com/why-meteor/features

Meteor Development Group. (2015, 12 15). _Introduction_. Retrieved from Meteor Docs: http://docs.meteor.com/

Ratkevic, M. (2015, 12 15). _Meteor Named A "Cool Vendor" by Gartner_. Retrieved from PR Newswire: http://www.prnewswire.com/news-releases/meteor-named-a-cool-vendor-by-gartner-300073337.html

Robinson , M. (2015, 12 15). _Why Node.js is Ideal for the Internet of Things_. Retrieved from ProgrammableWeb: http://www.programmableweb.com/news/why-node.js-ideal-internet-things/analysis/2014/07/31

Wikipedia. (2015, 12 15). _Wikipedia_. Retrieved from Web application: https://en.wikipedia.org/wiki/Web\_application

Zapponi, C. (2015, 12 15). _Language Discovery GitHub_. Retrieved from GitHut: http://githut.info/