---
title: Migrate Piwik website
date: 2014-04-24T07:23:42+00:00
author: Janik Vonrotz
slug: migrate-piwik-website
images:
  - /wp-content/uploads/2014/12/logo-piwik-e1418200408662.png
categories:
  - Web server
tags:
  - migration
  - mysql
  - piwik
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9640572](https://gist.github.com/9640572).*

# Introduction

This article assumes you're going to move an exisiting piwik website from one server to another.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [Nginx](https://janikvonrotz.ch/2014/03/31/install-nginx/)
* [Nginx minimal website](https://janikvonrotz.ch/2014/04/01/nginx-minimal-website/)
* [php5-fpm](https://janikvonrotz.ch/2014/03/20/install-php5-fpm/)
* [php5-mcrypt, php5-mysql](https://janikvonrotz.ch/2014/03/25/install-php5-modules/)
* [Nginx php5-fpm website](https://janikvonrotz.ch/2014/04/11/install-nginx-php5-fpm-website/)
* [MySQL](https://janikvonrotz.ch/2014/04/07/install-mysql/)
* [Increased Max Upload for php5-fpm website](https://janikvonrotz.ch/2014/04/11/increase-max-upload-for-php5-fpm-website/)
* [phpMyAdmin website](https://janikvonrotz.ch/2014/04/14/install-phpmyadmin-website/)
* [Piwik website](https://janikvonrotz.ch/2014/04/22/install-piwik-website/)

# Instructions

Export the existing SQL Piwik database with phpMyAdmin.

Import the database export into the new Piwik database with phpMyAdmin.

Update the tracking codes or update Piwik urls on the Piwik tracked websites if the hosting has changed.
  