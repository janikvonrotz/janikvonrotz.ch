---
title: Deploy Ubuntu server
date: 2014-03-13T16:04:21+00:00
author: Janik von Rotz
slug: deploy-ubuntu-server
dsq_thread_id:
  - "2421625589"
images:
  - /wp-content/uploads/2014/03/Ubuntu-Logo.png
categories:
  - Web server
tags:
  - aws
  - azure
  - deployment
  - private
  - project
  - rackspace
  - ubuntu
  - virtual
  - vps
---
<em>This post is part of my <a href="https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/">Your own Virtual Private Server hosting solution</a> project.</em>
<em>Get the latest version of this article here: <a href="https://gist.github.com/9468612">https://gist.github.com/9468612</a></em>.

<h1>Introduction</h1>

Why Ubuntu?

Because:

<ul>
<li>Ubuntu is the leading and most popular Linux distro. </li>
<li>It's easy to install, deploy and manage.</li>
<li>Ubuntu remains the most popular operating system for OpenStack deployments.</li>
<li>It's available for multiple devices.</li>
<li>Ubuntu benefits from a huge community support since ever.
<!--more--></li>
</ul>

<h1>Installation</h1>

I don't want to show you how to install an Ubuntu server, that's way to easy and this shouldn't really bother you.

The only decisions you have to make is wether to install it on your own or deploy it as an VPS (Virtual Private Server).

You can host an online VPS installation by several providers.

<ul>
<li><a href="https://aws.amazon.com/de/ec2/">Amazon AWS EC2</a></li>
<li><a href="https://www.rackspace.com/cloud/servers/">Rackspace Cloud Servers</a></li>
<li><a href="https://www.windowsazure.com/de-de/">Windows Azure</a></li>
</ul>

If you want to install it on your own on hardware or virtual machine, you can download it from the <a href="https://www.ubuntu.com/download/server">offical Ubuntu website</a>.

Unregarded of the installation and deployment strategy your server must...

<ul>
<li>...be secured with a dedicated firewall, that only allows http, https and ssh.</li>
<li>...being accessed with <a href="https://help.ubuntu.com/community/SSH/OpenSSH/Keys">ssh keys</a>.</li>
<li>...being accessed with an user without root privileges (only root substitution should be possible).</li>
</ul>

Personally I recommand to use the LTS (Long Term Support) version, compared to the latest releases they're more stable and secure.

<h1>Source</h1>

[Ubuntu server offical Website](https://www.ubuntu.com/server)