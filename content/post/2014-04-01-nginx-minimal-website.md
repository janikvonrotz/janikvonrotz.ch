---
title: Nginx minimal website
date: 2014-04-01T07:57:42+00:00
author: Janik von Rotz
slug: nginx-minimal-website
dsq_thread_id:
  - "2570589685"
image: /wp-content/uploads/2014/03/Nginx-Logo-e1394033855329.png
categories:
  - Nginx
tags:
  - configuration
  - nginx
  - server
  - template
  - website
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9408741](https://gist.github.com/9408741).*

# Introduction

This is a minimal Nginx website configuration. It's a good way to start your next project.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [Nginx](https://janikvonrotz.ch/2014/03/31/install-nginx/)

# Installation

Add this configuration file to the config folder `/etc/nginx/conf.d`.

    sudo vi /etc/nginx/conf.d/[host].conf


With the content showed below.

```
server{

    listen 80;
    server_name [host]

    root /var/www/[host];
    index index.php index.html;

    # host error and access log
    access_log /var/log/nginx/[host].access.log;
    error_log /var/log/nginx/[host].error.log;
    
    location / {
    }
    
    # redirect server error pages to the static page /50x.html
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
Update permissions for the `www-data` group.

    sudo chown www-data:www-data /var/www/[host] -R 
    
Test config and reload Nginx service.

    sudo nginx -t && sudo service nginx reload