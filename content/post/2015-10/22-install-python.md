---
title: Install Python
date: 2015-10-22T07:08:05+00:00
author: Janik von Rotz
slug: install-python
images:
  - /wp-content/uploads/2015/10/Python-Logo.png
categories:
  - Web server
tags:
  - python
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/739313df400616f999d6](https://gist.github.com/739313df400616f999d6).*

# Introduction

Configure Python 3 and install pip. Pip allows us to easily manage any Python 3 package we would like to have.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)

# Installation

First, check your current Python version.

    python --version

On a fresh Ubuntu server, this might output:

    Python 2.7.6

We would like to have python run Python 3. So first, let's remove the old 2.7 binary.

    sudo rm /usr/bin/python

Next, create a symbolic link to the Python 3 binary in its place.

    sudo ln -s /usr/bin/python3 /usr/bin/python

If you run `python --version` again, you will now see `Python 3.4.0`.

To install pip make sure update the aptitude repository index.

    sudo apt-get update

To install pip, simply run the following:

    sudo apt-get install python3-pip

# Source

[Install Python Lamp - Digital Ocean Community](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-apache-mysql-and-python-lamp-server-without-frameworks-on-ubuntu-14-04)
