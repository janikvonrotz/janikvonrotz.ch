---
id: 1825
title: Install automysqlbackup
date: 2014-04-08T08:33:11+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=1825
permalink: /2014/04/08/install-automysqlbackup/
dsq_thread_id:
  - "2595242917"
image: /wp-content/uploads/2014/04/MySQL-Logo.png
categories:
  - MySQL
tags:
  - auto
  - backup
  - easy
  - installation
  - mysql
  - reliable
  - simple
  - solution
  - ubuntu
---
*This post is part of my [Your own Virtual Private Server hosting solution](http://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9488132](https://gist.github.com/9488132).*  

# Introduction

AutoMySQLBackup with a basic configuration will create Daily, Weekly and Monthly backups of one or more of your MySQL databases from one or more of your MySQL servers.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [MySQL](https://janikvonrotz.ch/2014/04/07/install-mysql/)

# Installation

First create the backup folder.

    sudo mkdir /var/backups/mysql

Install the latest version of automysqlbackup from the [official website](http://sourceforge.net/projects/automysqlbackup).

    sudo mkdir /usr/local/src/automysqlbackup
    cd /usr/local/src/automysqlbackup
    sudo wget http://switch.dl.sourceforge.net/project/automysqlbackup/AutoMySQLBackup/AutoMySQLBackup%20VER%203.0/automysqlbackup-v3.0_rc6.tar.gz
    sudo tar xvzf automysqlbackup-v3.0_rc6.tar.gz
    sudo ./install.sh
    
Answer the install wizard as showed below.

    global configuration diectory: default
    directory for the executable: default
    
Add read permissios for all in the config folder.

    cd /etc/automysqlbackup
    sudo chmod a+r ./*
    
Update the automysqlbackup config file.

    sudo vi /etc/automysqlbackup/automysqlbackup.conf
    
Set credentials and update the backup path.

```
# Username to access the MySQL server e.g. dbuser
CONFIG_mysql_dump_username='root'

# Password to access the MySQL server e.g. password
CONFIG_mysql_dump_password='[password]'

# Backup directory location e.g /backups
CONFIG_backup_dir='/var/backups/mysql'
```

Unlock the latest backup in order to make backups for third party programs available

```
# Store an additional copy of the latest backup to a standard
# location so it can be downloaded by third party scripts.
CONFIG_mysql_dump_latest='yes'
```

Let's schedule the backup job by adding a daily cron script.

    sudo vi /etc/cron.daily/automysqlbackup
    
With the following content.

```
#!/bin/sh

/usr/local/bin/automysqlbackup /etc/automysqlbackup/automysqlbackup.conf

chown root.root /var/backup/mysql* -R
sudo chmod -R a-x+X /var/backup/mysql
```

Enable execution of the script for the owner.

    sudo chmod o+x /etc/cron.daily/automysqlbackup

You can run the backup script manually.

    cd /etc/cron.daily
    sudo ./automysqlbackup
    
Checkout the backup folder.

    cd /var//backup/mysql
    
# Source

[How To Backup MySQL Databases on an Ubuntu VPS by Digital Ocean](https://www.digitalocean.com/community/articles/how-to-backup-mysql-databases-on-an-ubuntu-vps)  
[How to back up MySQL databases using AutoMySQLBackup by A2 hosting](http://www.a2hosting.com/kb/developer-corner/mysql/mysql-database-backups-using-automysqlbackup)
[automysqlbackup website](http://sourceforge.net/projects/automysqlbackup/)  