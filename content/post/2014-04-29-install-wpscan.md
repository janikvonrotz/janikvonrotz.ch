---
id: 2126
title: Install WPScan
date: 2014-04-29T07:10:57+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=2126
permalink: /2014/04/29/install-wpscan/
dsq_thread_id:
  - "2646768728"
image: /wp-content/uploads/2014/04/wpscan_logo_407x80.png
categories:
  - IT Security
  - WordPress
tags:
  - bundler
  - exploit
  - gem
  - hardening
  - project
  - ruby
  - scan
  - scanner
  - security
  - vulnerability
  - wordpress
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/11214650](https://gist.github.com/11214650).*  

# Introduction

WPScan is a black box WordPress vulnerability scanner.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [libcurl4-gnutls-dev, libopenssl-ruby, libxml2, libxml2-dev, libxslt1-dev, ruby-dev](https://janikvonrotz.ch/2014/03/25/install-ubuntu-development-libraries/)
* [Git](https://janikvonrotz.ch/2014/03/25/install-ubuntu-packages/)
* [Ruby and RubyGems with RVM](https://janikvonrotz.ch/2014/04/28/install-ruby-and-rubygems-with-rvm/)

# Installation

First clone the WPScan repository from GitHub.

    cd /usr/local/src/
    sudo git clone https://github.com/wpscanteam/wpscan.git

Now install the bundler gem.

    sudo chown [current username]:[current username] wpscan/
    cd wpscan/
    gem install bundler
    
Install the WPScan project with user priviliges.
    
    bundle install --without test

Run a scan.

    ruby wpscan.rb --url [url]

# Source

[WPScan Github Repository](https://github.com/wpscanteam/wpscan)