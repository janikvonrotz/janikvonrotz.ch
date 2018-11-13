---
title: Install php5-fpm
date: 2014-03-20T13:41:30+00:00
author: Janik von Rotz
slug: install-php5-fpm
images:
  - /wp-content/uploads/2014/03/php-logo.jpeg
categories:
  - Web server
tags:
  - installation
  - php
  - ubuntu
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9343960](https://gist.github.com/9343960).*  
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)

# Installation

Install the package with aptitude.

    sudo aptitude install php5-fpm
    
Check the php5-fpm version.

    php5-fpm -v

# Source

[Configuring and Optimizing PHP-FPM and Nginx on Ubuntu](http://blog.chrismeller.com/configuring-and-optimizing-php-fpm-and-nginx-on-ubuntu-or-debian)