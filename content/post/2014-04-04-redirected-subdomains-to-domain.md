---
id: 1784
title: Redirected subdomains to domain
date: 2014-04-04T06:59:12+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=1784
permalink: /2014/04/04/redirected-subdomains-to-domain/
dsq_thread_id:
  - "2585121297"
image: /wp-content/uploads/2014/03/Nginx-Logo-e1394033855329.png
categories:
  - Nginx
tags:
  - canonical
  - domain
  - module
  - nginx
  - redirect
  - rewrite
  - subdomain
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9433480](https://gist.github.com/9433480).*

# Introduction

A website has to be only accessible by one specific host name. Otherwise search engines will index your website two or more times.
A lot of people think that a website should only be published as `www.[host]`.
I think this is wrong, using the `www` section is a old fashioned way of how companies have structured their DNS records.
There are users which still use the `www` before tipping an url and others who don't.
However we will allow both of them to access our website.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [Nginx](https://janikvonrotz.ch/2014/03/31/install-nginx/)
* [Nginx minimal website](https://janikvonrotz.ch/2014/04/01/nginx-minimal-website/)

# Installation

Redirecting every possible subdomain f.g. containing www.example.org is easily redirected to a prefrered url by adding the following Nginx configuration to the host config file.

```
server{

    server_name *.[host];
    
    return 301 http://[host]$request_uri;
}
    
```
Test config and reload Nginx service.

    sudo nginx -t && sudo service nginx reload

# Source

[Nginx server names](http://nginx.org/en/docs/http/server_names.html)  
[Nginx converting rewrite rules](http://nginx.org/en/docs/http/converting_rewrite_rules.html)  