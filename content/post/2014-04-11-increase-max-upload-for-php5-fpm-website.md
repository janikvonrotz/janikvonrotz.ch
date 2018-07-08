---
title: Increase Max Upload for php5-fpm website
date: 2014-04-11T06:55:10+00:00
author: Janik von Rotz
permalink: /2014/04/11/increase-max-upload-for-php5-fpm-website/
dsq_thread_id:
  - "2603118431"
image: /wp-content/uploads/2014/03/php-logo.jpeg
categories:
  - PHP
tags:
  - applications
  - fpm
  - increase
  - max
  - nginx
  - php
  - php5
  - ubuntu
  - upload
  - website
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9604715](https://gist.github.com/9604715).*  

# Introduction

In some cases the default memory allocation for php is not enough to run an application properly.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [Nginx](https://janikvonrotz.ch/2014/03/31/install-nginx/)
* [Nginx minimal website](https://janikvonrotz.ch/2014/04/01/nginx-minimal-website/)
* [php5-fpm](https://janikvonrotz.ch/2014/03/20/install-php5-fpm/)
* [Nginx php5-fpm website](https://janikvonrotz.ch/2014/04/11/install-nginx-php5-fpm-website/)

# Instruction

Update the php5-fpm config.

    sudo vi /etc/php5/fpm/php.ini
    Set memory_limit = 512M

Restart the php5-fpm service.

    sudo service php5-fpm restart
    