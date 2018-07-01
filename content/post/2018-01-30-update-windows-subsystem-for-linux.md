---
id: 4764
title: Update Windows Subsystem for Linux
date: 2018-01-30T10:32:51+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4764
permalink: /2018/01/30/update-windows-subsystem-for-linux/
specific_page_layout:
  - default-sidebar
image: /wp-content/uploads/2018/01/Bash-on-Ubuntu-on-Windows.png
categories:
  - Ubuntu
  - Windows
tags:
  - linux
  - packages
  - trusty
  - ubuntu
  - ugly hack
  - update
  - version
  - windows subsystem for linux
  - wsl
  - xenial
---
Today I learned that certain Ubuntu package versions are bound to the release version of Ubuntu. For example the only available version of the password store tool *pass* for Ubuntu LTS 14.04 (trusty) is *[1.4.2-3](https://packages.ubuntu.com/trusty/admin/pass)*. If you need a newer version you have to update Ubuntu first. Usually this no big deal, however, if you work with Windows Subsystem for Linux (WSL) it is a big deal. The WSL release is bound to the Windows version. In result to update a package you have to update your Windows first.
<!--more-->

## Common Solution

In case your Windows version is newer than your WSL version uninstall WSL.

`lxrun /uninstall /full`

And reinstall WSL.

`lxrun /install`

Once finished you can check the version with `lsb_release -a`.

## Ugly Hack

If you work in corporate environment you might not be able to get the latest Windows release. In this case you have to update Ubuntu itself which is not supported by Microsoft.

First we have to prevent certain packages from being updated.

`sudo -S apt-mark hold procps strace sudo`

These commands have been modified especially for WSL.

Next run the release upgrade command.

`sudo -S env RELEASE_UPGRADER_NO_SCREEN=1 do-release-upgrade`

If the update successfully finished you should be on the latest release.