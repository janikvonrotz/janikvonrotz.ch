---
title: Install Nginx php5-fpm website
date: 2014-04-11T06:52:52+00:00
author: Janik von Rotz
slug: install-nginx-php5-fpm-website
dsq_thread_id:
  - "2603114610"
image: /wp-content/uploads/2014/03/Nginx-Logo-e1394033855329.png
categories:
  - Nginx
  - PHP
tags:
  - application
  - configuration
  - minimal
  - nginx
  - php
  - php5
  - website
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9445746](https://gist.github.com/9445746).*  

# Introduction

This is a minimal Nginx configuration to run php based websites/ applications.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [Nginx](https://janikvonrotz.ch/2014/03/31/install-nginx/)
* [Nginx minimal website](https://janikvonrotz.ch/2014/04/01/nginx-minimal-website/)
* [php5-fpm](https://janikvonrotz.ch/2014/03/20/install-php5-fpm/)

# Installation

Add this Nginx configuration to your website config.
```
server{

    ...
    
    # php5-fpm configuration
    location ~ \.php$ {
        
        set $php_root /var/www/[host];
        
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $php_root$fastcgi_script_name;
        include /etc/nginx/fastcgi_params;
    }
}
```
Test config and reload Nginx service.

    sudo nginx -t && sudo service nginx reload