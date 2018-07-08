---
id: 1804
title: Install MySQL
date: 2014-04-07T07:14:46+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=1804
permalink: /2014/04/07/install-mysql/
dsq_thread_id:
  - "2592456287"
image: /wp-content/uploads/2014/04/MySQL-Logo.png
categories:
  - MySQL
tags:
  - aptitude
  - database
  - installation
  - most
  - mysql
  - popular
  - secure
  - system
  - ubuntu
  - version
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9392658](https://gist.github.com/9392658).*

# Introduction

MySQL is the world's most popular open source database system.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)

# Installation

Install MySQL server

    sudo apt-get install mysql-server
    
Set the mysql root user password during the installation

Install the default MySQL databases

    sudo mysql_install_db
    
Run the finisher script and respond except for the first prompt with yes in order to get a secure MySQL installation

    sudo /usr/bin/mysql_secure_installation
        
Connect to your new MySQL server

    mysql -uroot -p
    
Enter the root password

And run this command to get the MySQL version

    SHOW variables LIKE "%version%";
	
# Source

[Ubuntu MySQL server guide](https://help.ubuntu.com/12.04/serverguide/mysql.html)