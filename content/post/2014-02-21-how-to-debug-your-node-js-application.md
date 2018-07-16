---
title: How to debug your Node.js application
date: 2014-02-21T14:20:13+00:00
author: Janik von Rotz
slug: how-to-debug-your-node-js-application
dsq_thread_id:
  - "2298328026"
image: /wp-content/uploads/2014/02/nodejs-2560x1440-e1394184527626.png
categories:
  - Blog
  - Node.js
tags:
  - app
  - debugging
  - editor
  - inspector
  - javascript
  - node.js
---
Recently I've started developing with Node.js. As a beginner I had to set up an <a href="https://en.wikipedia.org/wiki/Integrated_Development_Environment">IDE</a> which meets my requirements such as debugging code. I've tried several IDE's such as Microsofts's WebMatrix or Eclipse Node.js extension. But all of them literally sucked, they were are a pain to install or didn't work properly.

Luckily I came along this project: <a href="https://github.com/node-inspector/node-inspector">https://github.com/node-inspector/node-inspector</a>

<!--more-->

It's a Node.js debugger that runs in your browser! That means you can still use your the code editor you're into and debugging the running code directly in your browser (chrome or opera so far). That's totally awesom! Isn't it?

To get started with node inspector install these tools:

<ul>
    <li>Node.js (of course with npm)</li>
    <li>A text editor you like f.g. Nodepad++ or Sublime</li>
    <li>A Node.js project (I'll use the express framework)</li>
</ul>

To install node inspector run these commands in your project folder from the command line:

```js

// install node inspector globally as developer dependencies

npm install -g node-inspector --save-dev

// start debugging you app

node-debug app.js

```

Your browser will open the developer console on the debugging page now.

[![node inspector example](/wp-content/uploads/2014/02/node-inspector-example.jpg)](/wp-content/uploads/2014/02/node-inspector-example.jpg)PS: It is even possible to edit the code live in the browser!