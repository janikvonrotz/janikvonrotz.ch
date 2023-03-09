---
title: Node.js Nginx proxy website
date: 2014-04-02T06:52:03+00:00
author: Janik von Rotz
slug: node-js-nginx-proxy-website
images:
  - /wp-content/uploads/2014/03/Node.js-Logo.png
categories:
  - Web server
tags:
  - application
  - javascript
  - nginx
  - nodejs
  - proxy
  - security
  - website
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9407504](https://gist.github.com/9407504).*

# Introduction

It's recommanded to publish a Node.js application with a Nginx proxy website.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [Node.js](https://janikvonrotz.ch/2014/03/27/install-node/)
* [Nginx](https://janikvonrotz.ch/2014/03/31/install-nginx/)
* [Nginx minimal website](https://janikvonrotz.ch/2014/04/01/nginx-minimal-website/)

# Installation

Add this Nginx config to one of your website.
```
server {

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:[port];
    }
}
```
Where `proxy_pass` port is the must be equal with the port of the Node.js application.

Restart the Nginx service.

    sudo service nginx restart