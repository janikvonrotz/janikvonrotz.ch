---
title: Install Ubuntu development libraries
date: 2014-03-25T11:42:14+00:00
author: Janik Vonrotz
slug: install-ubuntu-development-libraries
images:
  - /wp-content/uploads/2014/03/Ubuntu-Logo.png
categories:
  - Web server
tags:
  - aptitude
  - development
  - libaries
  - package
  - ubuntu
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9758712](https://gist.github.com/9758712).*  

# Introduction

This is a list of essential tools for developing with Ubuntu.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)

# Installation

## build-essential

This package contains an informational list of packages which are considered essential for building Debian packages.

    sudp apg-get install build-essential

## libcurl4-gnutls-dev

These files (ie. includes, static library, manual pages) allow to build software which uses libcurl.

    sudo apt-get install libcurl4-gnutls-dev

## libgeoip-dev

GeoIP is a C library that enables the user to find the country that any IP address or hostname originates from. It uses a file based database.

    sudo apt-get install libgeoip-dev
    
## libopenssl-ruby

This package makes Ruby to be able to use OpenSSL.

    sudo apt-get install libopenssl-ruby

## libxml2

This package provides a library providing an extensive API to handle XML data files.

    sudo apt-get install libxml2
    
## libxml2-dev

Install this package if you wish to develop your own programs using the GNOME XML library.

    sudo apt-get install libxml2-dev
    
## libxslt1-dev

This package contains the development files libxslt.

    sudo apt-get install libxslt1-dev

## ruby-dev

This package provides the header files, necessary to make extension library for Ruby.

    sudo apt-get install ruby-dev