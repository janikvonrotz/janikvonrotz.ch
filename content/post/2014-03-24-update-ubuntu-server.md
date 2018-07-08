---
title: Update Ubuntu server
date: 2014-03-24T18:34:00+00:00
author: Janik von Rotz
permalink: /2014/03/24/update-ubuntu-server/
dsq_thread_id:
  - "2500275221"
image: /wp-content/uploads/2014/03/Ubuntu-Logo.png
categories:
  - Ubuntu Server
tags:
  - all
  - aptitude
  - os
  - package
  - server
  - ubuntu
  - update
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9384311](https://gist.github.com/9384311).* 

# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
<!--more-->
# Instructions

Update the list of available packages.

    sudo aptitude update
    
Upgrade any that have updates available.

    sudo apt-get dist-upgrade