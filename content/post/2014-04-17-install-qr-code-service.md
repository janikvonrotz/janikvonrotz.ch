---
id: 2004
title: Install QR code service
date: 2014-04-17T10:35:03+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=2004
permalink: /2014/04/17/install-qr-code-service/
dsq_thread_id:
  - "2618446085"
image: /wp-content/uploads/2014/04/QR-Code-service.png
categories:
  - Nginx
  - PHP
tags:
  - code
  - configuration
  - fpm
  - nginx
  - php
  - php5
  - qr
  - service
  - ubuntu
  - website
---
*This post is part of my [Your own Virtual Private Server hosting solution](http://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9445729](https://gist.github.com/9445729).*  

# Introduction

To generate QR codes with php we are using the project [QR Generator PHP](https://github.com/janikvonrotz/QR-Generator-PHP) hosted on GitHub.
<!--more-->


# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [Nginx](https://janikvonrotz.ch/2014/03/31/install-nginx/)
* [Nginx minimal website](https://janikvonrotz.ch/2014/04/01/nginx-minimal-website/)
* [php5-fpm](https://janikvonrotz.ch/2014/03/20/install-php5-fpm/)
* [Nginx php5-fpm website](https://janikvonrotz.ch/2014/04/11/install-nginx-php5-fpm-website/)

# Installation

Clone project with git.

    cd /usr/local/src
    sudo git clone https://github.com/janikvonrotz/QR-Generator-PHP.git

Rename the project directory to get a shorter url.

    sudo mv QR-Generator-PHP qr

Add the config to one of your Nginx sites.


```
server{
    
    ...
    
    # change location for QR code requests
    location /qr{
        root /usr/local/src;
        index index.php;
    }
    
    location ~ .php$ {
        
        ...
        
        # change the php root for QR code requests
        if ($request_uri ~* /qr) {
            set $php_root /usr/local/src;
        }
        
        ...
    }
}
```

Test config and reload Nginx service.

    sudo nginx -t && sudo service nginx reload

Test the new QR code service by open a browser on `//[host]/qr/?d=example.org`.