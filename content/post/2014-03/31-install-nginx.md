---
title: Install Nginx
date: 2014-03-31T09:43:19+00:00
author: Janik von Rotz
slug: install-nginx
images:
  - /wp-content/uploads/2014/03/Nginx-Logo-e1394033855329.png
categories:
  - Web server
tags:
  - hosting
  - nginx
  - pgp
  - web server
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9345486](https://gist.github.com/9345486).*  

# Introduction

Nginx (pronounced "engine-ex") can be used as an open source reverse proxy server, as well as a load balancer, HTTP cache, and not to forget a fast and powerful web server.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)

# Installation

As the version of Nginx that comes with Ubuntu can be incredibly outdated we add the Nginx project repository.

Add the PGP packge key.

	wget -O - http://nginx.org/keys/nginx_signing.key | sudo apt-key add -
	
Get the release code name of your Ubuntu installation.

	cat /etc/lsb-release
		
Where `DISTRIB_CODENAME` is the release codename.

Or get the release code name of your Debian installation.

	cat /etc/os-release
		
Where `VERSION` is the release codename.

Let's a new source list for nginx. 

    sudo vi /etc/apt/sources.list.d/nginx.list
	
Add the following two line for an Ubuntu codename, f.g. mine was "precise".

    deb http://nginx.org/packages/ubuntu/ [codename] nginx
    deb-src http://nginx.org/packages/ubuntu/ [codename] nginx
	
Add the following two line for a Debian codename, f.g. mine was "squeeze".

    deb http://nginx.org/packages/debian/ [codename] nginx
    deb-src http://nginx.org/packages/debian/ [codename] nginx
	
If you have installed an old version of Nginx (such as your distro’s native version) you can get rid of it quickly by using this command.

    sudo aptitude purge nginx nginx-light nginx-full nginx-extras nginx-common
	
Be aware the the command purge will also delete all related config files.

Update the package list and install Nginx.

    sudo aptitude update
    sudo aptitude install nginx
	
Check nginx version.

    nginx -v

Add the nginx user to the built-in www-data group.

    sudo usermod -a -G www-data nginx

# Source

[Configuring and Optimizing PHP-FPM and Nginx on Ubuntu](http://blog.chrismeller.com/configuring-and-optimizing-php-fpm-and-nginx-on-ubuntu-or-debian)