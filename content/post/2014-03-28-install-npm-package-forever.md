---
title: Install npm package forever
date: 2014-03-28T12:49:20+00:00
author: Janik von Rotz
permalink: /2014/03/28/install-npm-package-forever/
dsq_thread_id:
  - "2532782991"
image: /wp-content/uploads/2014/03/Node.js-Logo.png
categories:
  - Node.js
tags:
  - abort
  - applications
  - forever
  - installation
  - interruption
  - manager
  - node
  - npm
  - package
  - run
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9347463](https://gist.github.com/9347463).*  

# Introduction

A simple CLI tool for ensuring that a given script runs continuously (i.e. forever).
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [Node.js](https://janikvonrotz.ch/2014/03/27/install-node/)

# Installation

Install forever

    sudo npm install forever -g
	
Start Node.js application with forever

	sudo NODE_ENV=production forever start index.js
    
Start Node.js application without forever

	sudo npm start --production
	
List Node.js applications executed by forever

	forever list
	
Stop forever applications

	forever stop index.js