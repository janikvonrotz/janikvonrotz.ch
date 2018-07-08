---
title: Install Ruby and RubyGems with RVM
date: 2014-04-28T10:21:50+00:00
author: Janik von Rotz
permalink: /2014/04/28/install-ruby-and-rubygems-with-rvm/
dsq_thread_id:
  - "2644399290"
image: /wp-content/uploads/2014/04/Ruby-Logo.png
categories:
  - Ruby
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*
*Get the latest version of this article here: [https://gist.github.com/11215397](https://gist.github.com/11215397).*

# Introduction

Ruby is a dynamic, open source programming language with a focus on simplicity and productivity.
To install Ruby we are going to use RVM (Ruby Version Manager).
This installation differs from other as you don't run it with user privilegs, that means no more sudoing for this guide.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)

# Installation

First install RVM with the latest stable release version of Ruby by fetching the RVM install script with curl.

    \curl -sSL https://get.rvm.io | bash -s stable --ruby

After the installation add you current user to the rvm group.

    sudo adduser [username] rmv
    
For the same user add the rvm startup script to the bash profile.

    echo "source $HOME/.rvm/scripts/rvm" >> ~/.bash_profile
    
Now log out from the cli and log in to load RVM properly.

Check the installed RVM version.

    rvm -v

As RubyGems comes as built-in extension to RVM we don't need to install it.

# Source

[Installing RVM](http://rvm.io/rvm/install)
[Setup Ruby On Rails on Ubuntu 14.04 Trusty Tahr](https://gorails.com/setup/ubuntu/14.04)  
[Ruby GitHub repository](https://github.com/ruby/ruby)
[RubyGems download website](https://rubygems.org/pages/download)