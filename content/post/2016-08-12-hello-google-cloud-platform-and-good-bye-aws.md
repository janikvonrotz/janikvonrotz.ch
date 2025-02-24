---
title: Hello Google Cloud Platform and good bye AWS
date: 2016-08-12T14:41:54+00:00
author: Janik von Rotz
slug: hello-google-cloud-platform-and-good-bye-aws
images:
  - /wp-content/uploads/2016/08/Google-Cloud-Platform-1200x741.png
categories:
  - Web server
  - System engineering
  - Cloud computing
  - Blog
tags:
  - google
  - hosting
---
Yesterday late afternoon I had the great idea to update my AWS EC2 instance to Ubuntu 16.04 LTS. This website and two other sites are hosted on this machine. A Piwik installation and a mail forwarder as well. So it's not that much, but still very [essential to me](http://fusion.net/story/325231/google-deletes-dennis-cooper-blog/). The upgrade didn't go well, the system literally broke.
<!--more-->
![Ubuntu update 16.04 LTS fail](/wp-content/uploads/2016/08/Ubuntu-update-16.04-LTS-fail.png)

I felt devastated because I didn't really know what to do next and went to sleep. After six hours of sleep later I've decided to setup a new web server. Either with Microsoft Azure or Google Cloud Platform. The only requirement was simplicity. Whichever felt more intuitive to install an Ubuntu server 16.04 with Nginx, Mysql, Php, Wordpress and Piwik will be the winner. I don't want to go in details here, but obviously the winner was GCP and I still don't how the Azure dashboard works. I was very surprised by the slick setup process. Didn't had to care about adding firewall rules for http and https (just a check box tick) or binding a static IP address, it simply worked as expected. Four hours later my sites are back and running. Amazing isn't it? This process would have been a real pain a few years ago. Currently I'm quite happy with GCP, however, let's see the first bill. I will keep you updated.