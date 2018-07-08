---
id: 1699
title: Install php5 modules
date: 2014-03-25T11:38:33+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=1699
permalink: /2014/03/25/install-php5-modules/
dsq_thread_id:
  - "2505352611"
image: /wp-content/uploads/2014/03/php-logo.jpeg
categories:
  - PHP
  - Ubuntu Server
tags:
  - essential
  - fpm
  - install
  - list
  - modules
  - php
  - server
  - ubuntu
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9758559](https://gist.github.com/9758559).*  

# Introduction

Php5 modules extending the functionality for php scripts and are required by several php projects.
<!--more-->
# Requirements

* Ubuntu server
* php5-fpm

# Installation

Restart the php5-fpm service after every installation.

    sudo service php5-fpm restart

## php5-curl

CURL is a library for getting files from FTP, GOPHER, HTTP server.

    sudo apt-get install php5-curl

## php5-dev

This package provides the files from the PHP5 source needed for compiling additional modules.

    sudo apt-get install php5-dev 
	
## php5-geoip

This PHP module allows you to find the location of an IP address - City, State, Country, Longitude, Latitude, and other information as all, such as ISP and connection type.

    sudo apt-get install php5-geoip
	
## php5-mcrypt

This package provides a module for MCrypt functions in PHP scripts.

    sudo apt-get install php5-mcrypt

## php5-mysql

This package provides modules for MySQL database connections directly from PHP scripts.

    sudo apt-get install php5-mysql
	
# Source
	
[Ubuntu package repository](http://packages.ubuntu.com/)