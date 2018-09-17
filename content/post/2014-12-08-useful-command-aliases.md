---
title: Useful command aliases
date: 2014-12-08T10:04:00+00:00
author: Janik von Rotz
slug: useful-command-aliases
dsq_thread_id:
  - "3301837297"
images:
  - /wp-content/uploads/2014/03/Ubuntu-Logo.png
categories:
  - Ubuntu Server
tags:
  - aliases
  - command
  - line
  - linux
  - ubuntu
  - useful
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/5ac7998b00900a3680d7](https://gist.github.com/5ac7998b00900a3680d7).* 

# Introduction

This is a list of useful commandline aliases for your Ubuntu installation.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* Supported installations (see below)

# Instructions

The structure of the command aliases is a mix of the first 3 letters of the programm you're running and the parameters you're adding.
To set this aliases permanently add them to bash profile scirpt.

    vi ~/.bashrc

## duplicity-backup

List current files from your latest backup.

    alias duplcf="sudo /usr/local/src/duplicity-backup/duplicity-backup.sh -c /etc/duplicity-backup/duplicity-backup.conf --list-current-files"

Run a backup.

    alias dupbak="sudo /usr/local/src/duplicity-backup/duplicity-backup.sh -c /etc/duplicity-backup/duplicity-backup.conf -b"

Get the status of the latest backup.

    alias dupsta="sudo /usr/local/src/duplicity-backup/duplicity-backup.sh -c /etc/duplicity-backup/duplicity-backup.conf -s"