---
id: 42
title: 'Vagrant &#8211; portable development environment'
date: 2013-07-02T11:54:35+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=42
permalink: /2013/07/02/vagrant-portable-development-environment/
dsq_thread_id:
  - "1458874559"
image: /wp-content/uploads/2013/07/vagrant-logo.png
categories:
  - Vagrant
tags:
  - code
  - machine
  - portable
  - vagrant
  - virtual
---
Today almost every coder uses git or another vcs and a related service like github for code hosting.

As a front end designer twitters bower is the right tool to manage your libary dependencies.

And for the backend developer the npm package manager is always on board.

Those tools have changed the way of developing websites and webapplication.

They've increased productivity and the flexibilty for your application.

So far, there's only one part lagging behind: It's the development environment.

<!--more-->

And yes, also for this part exists a solution. I'm talking about <a href="https://www.vagrantup.com/">vagrant</a>

It is the lost child in my development workflow, it's the perfect supplement.

With vagrant and vm provider of your choice (virtualbox, vmware or amazon so far) you're abble to setup a portable and reproducible development environment. Vargant installs a virtual machine based on virtual appliance, by default vargant syncs your folder with the virtual machines default directory. With this setup you are able to run your code in virtual machine but still code your stuff as it appeals to you on your local host.

Well, that looks interesting, where should I start? For newbies the offical documentation should be fine: <a href="https://docs.vagrantup.com/v2/getting-started/index.html">https://docs.vagrantup.com/v2/getting-started/index.html</a>

Follow step by step and you'll run a virtual linux appliance with an apache preinstalled.