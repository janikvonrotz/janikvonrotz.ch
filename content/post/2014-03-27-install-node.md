---
title: Install Node
date: 2014-03-27T16:46:14+00:00
author: Janik von Rotz
slug: install-node
dsq_thread_id:
  - "2525008273"
image: /wp-content/uploads/2014/03/Node.js-Logo.png
categories:
  - Node.js
tags:
  - git
  - install
  - installation
  - latest
  - newest
  - node
  - nodejs
  - repo
  - ubuntu
  - version
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9346588](https://gist.github.com/9346588).*  

# Introduction

Node.js is a cross-platform runtime environment for server-side and networking applications.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [build-essential](https://janikvonrotz.ch/2014/03/25/install-ubuntu-development-libraries/)
* [Git](https://janikvonrotz.ch/2014/03/25/install-ubuntu-packages/)

# Installation

You can install Node.js either from website or the from the git repo.

## Install from source
	
Download Node.js with wget.

    cd /usr/local/src
    sudo wget http://nodejs.org/dist/node-latest.tar.gz

Unpack Node.js.

    sudo tar -xzf node-latest.tar.gz
    cd node-<version>
	
Install Node.js.

    sudo ./configure
    sudo make
    sudo make install
	
Check version of Node.js and npm.
	
    node -v
    npm -v

## Install with Git

Clone the Node.js repo.

	cd /usr/local/src
	sudo git clone git://github.com/joyent/node.git
	
Or use the https url if ssh is not possible.

  sudo git https://github.com/joyent/node.git

Check git tags to find the latest version.

	cd node
	git tag
	
See the latest stable version on [http://nodejs.org/](http://nodejs.org/).

Checkout the latest version.

	sudo git checkout vX.X.X
	
Install Node.js.

	sudo ./configure
	sudo make
	sudo make install

Check version of Node.js and npm.
	
	node -v
	npm -v

# Update

Depending on how you've installed Node.js theres an update strategy.

## from source

Repeat the installation process above.

## with Git

Pull down the latest source code.

	cd /usr/local/src/node
	sudo git checkout master
	sudo git pull origin master
	
Check git tags to find the latest version.

	git tag
	
See the latest stable version on [http://nodejs.org/](http://nodejs.org/).
	
Compile the latest version.

	sudo git checkout vx.x.x
	sudo ./configure
	sudo make
	sudo make install