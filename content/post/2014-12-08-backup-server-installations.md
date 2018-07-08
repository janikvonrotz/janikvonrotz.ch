---
title: Backup server installations
date: 2014-12-08T10:04:07+00:00
author: Janik von Rotz
slug: backup-server-installations
dsq_thread_id:
  - "3301838779"
image: /wp-content/uploads/2014/03/Ubuntu-Logo.png
categories:
  - Ubuntu Server
tags:
  - backup
  - files
  - folders
  - installation
  - recommanded
  - ubuntu
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/676d3ab77d9d48cba73e](https://gist.github.com/676d3ab77d9d48cba73e).* 

# Introduction

This article shows a list of the most important files and folders of your VPC installations, you definitely should backup.
It assumes you'll run a daily backup for this ressources.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* Supported installations (see below)

# Instructions

As long new versions of your installations are released frequently, it's recommanded to backup data and configuration files only.

## Ubuntu

    -
    
## duplicity-backup

    /etc/duplicity-backup/*

## Koken

    /var/www/<koken>

## MySQL

If you have automysqlbackup running.

    /var/backups/mysql/latest/*
    
## Nginx

    /etc/nginx/conf.d/*  

## Postfix

    /etc/postfix/virtual
    /etc/postfix/main.cf

## Piwik

    /var/www/<piwik>/config/*
    
## WordPress

    /var/www/<wordpress>/wp-config.php
    /var/www/<wordpress>/wp-content/*