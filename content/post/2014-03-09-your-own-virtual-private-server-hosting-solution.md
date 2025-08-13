---
title: Your own Virtual Private Server hosting solution
date: 2014-03-09T11:16:23+00:00
author: Janik von Rotz
slug: 
aliases: your-own-virtual-private-server-hosting-solution
  - /your-own-virtual-private-server-hosting-solution/
  - /projects/your-own-virtual-private-server-hosting-solution/
images:
  - /wp-content/uploads/2014/03/Ubuntu-Logo.png
categories:
  - Web server
tags:
  - private
  - virtual server
  - hosting
  - guide
---
VPS (Virtual Private Server) hosting is the next level up from shared hosting.
Due to the explosive progress in cloud computing now everybody can afford a virtual private server.

Compared to a conventional cloud hosting solution with VPS you'll gain full control of your data.
With VPS you are responsible that your data is secured, available and always backed up.

There are many hosters which offer a VPS soution. However the bigger the company in this business the cheaper is their VPS service.

Just to name some of them:

<ul>
<li><a href="https://aws.amazon.com/de/ec2/">Amazon AWS EC2</a></li>
<li><a href="https://www.rackspace.com/cloud/servers/">Rackspace Cloud Servers</a></li>
<li><a href="https://www.windowsazure.com/de-de/">Windows Azure</a></li>
</ul>

With this project I want to show how easy it is to set up a Virtual Private Server hosting solution.

Divided in different posts I'm going to install a secure and reliable Ubuntu server. Equipped with different service such as a WordPress website, Node.js installation or a Nginx SSL secured proxy website.

<h1>Requirements</h1>

Your server must...

<ul>
<li>...be secured with a dedicated firewall, that only allows http, https and ssh.</li>
<li>...being accessed with <a href="https://help.ubuntu.com/community/SSH/OpenSSH/Keys">ssh keys</a>.</li>
</ul>

You have to be able ...

<ul>
<li>...to use the linux/unix command line.</li>
<li>...to edit files with <a href="http://www.cheatography.com/ericg/cheat-sheets/vi-editor/">VI</a></li>
</ul>

<h1>Modules</h1>

The installation of the VPS is built up according to a modular system. Every module corresponds to a blog post with their own requirements (mostly other modules) and installation instructions.

## Basic

**Server installation**

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [Update Ubuntu Server](https://janikvonrotz.ch/2014/03/24/update-ubuntu-server/)
* [SSH and network hardening](https://janikvonrotz.ch/2014/03/21/ssh-and-network-hardening/)

**Development libaries**

* [build-essential, libcurl4-gnutls-dev, libgeoip-dev, libopenssl-ruby, libxml2, libxml2-dev, libxslt1-dev, ruby-dev](https://janikvonrotz.ch/2014/03/25/install-ubuntu-development-libraries/)

**Essential programs**

* [dnsmasq, duplicity, Git, GnuPG, NcFTP, rng-tools, unzip](https://janikvonrotz.ch/2014/03/25/install-ubuntu-packages/)
* [Python](https://janikvonrotz.ch/2015/10/22/install-python/)

**Nginx**

* [Nginx](https://janikvonrotz.ch/2014/03/31/install-nginx/)
* [Nginx minimal website](https://janikvonrotz.ch/2014/04/01/nginx-minimal-website/)
* [Redirected subdomains to domain](https://janikvonrotz.ch/2014/04/04/redirected-subdomains-to-domain/)

**PHP**

* [php5-fpm](https://janikvonrotz.ch/2014/03/20/install-php5-fpm/)
* [Nginx php5-fpm website](https://janikvonrotz.ch/2014/04/11/install-nginx-php5-fpm-website/)
* [php5-curl, php5-dev, php5-geoip, php5-mcrypt, php5-mysql](https://janikvonrotz.ch/2014/03/25/install-php5-modules/)
* [Increase Max Upload for php5-fpm website](https://janikvonrotz.ch/2014/04/11/increase-max-upload-for-php5-fpm-website/)

## Advanced

**HHVM**

* [HHVM](https://janikvonrotz.ch/2014/04/30/install-hhvm/)

**SSL**

* [Get a free verified SSL certificate from StartSSL](https://janikvonrotz.ch/2014/03/26/get-a-free-verified-ssl-certificate-from-startssl/)
* [Convert SSL certificates](https://janikvonrotz.ch/2014/03/27/convert-ssl-certificates/)
* [Nginx SSL website](https://janikvonrotz.ch/2014/04/03/nginx-ssl-website/)
* [Install Let’s Encrypt and create a free SSL certificate](https://janikvonrotz.ch/2015/12/04/install-lets-encrypt-and-create-a-free-ssl-certificate/)
* [Configure Let’s Encrypt auto renewal for certificates](https://janikvonrotz.ch/2016/02/14/configure-lets-encrypt-auto-renewal-for-certificates/)

**Node.js**

* [Node.js](https://janikvonrotz.ch/2014/03/27/install-node/)
* [npm package forever](https://janikvonrotz.ch/2014/03/28/install-npm-package-forever/)
* [Node.js Nginx proxy website](https://janikvonrotz.ch/2014/04/02/node-js-nginx-proxy-website/)

**Ruby**

* [Ruby and RubyGems with RVM](https://janikvonrotz.ch/2014/04/28/install-ruby-and-rubygems-with-rvm/)

**Postfix**

* [Postfix with mail forwarding](https://janikvonrotz.ch/2014/12/05/install-postfix-with-mail-forwarding/)

**MySQL**

* [MySQL](https://janikvonrotz.ch/2014/04/07/install-mysql/)
* [automysqlbackup](https://janikvonrotz.ch/2014/04/08/install-automysqlbackup/)

## Expert

**Encryption**

* [GPG Keys](https://janikvonrotz.ch/2014/04/09/create-gpg-keys/)

**Services**

* [phpMyAdmin website](https://janikvonrotz.ch/2014/04/14/install-phpmyadmin-website/)
* [QR Code service](https://janikvonrotz.ch/2014/04/17/install-qr-code-service/)

**WordPress**

* [WordPress website](https://janikvonrotz.ch/2014/04/15/install-wordpress-website/)
* [Migrated WordPress website](https://janikvonrotz.ch/2014/04/16/migrate-wordpress-website/)
* [WPScan](https://janikvonrotz.ch/2014/04/29/install-wpscan/)

**Piwik**

* [Piwik website](https://janikvonrotz.ch/2014/04/22/install-piwik-website/)
* [Piwik geolocation support with GeoIP PECL](https://janikvonrotz.ch/2014/04/23/enable-piwik-geolocation-support-with-geoip-pecl/)
* [Migrate Piwik website](https://janikvonrotz.ch/2014/04/24/migrate-piwik-website/)

**LimeSurvey**

* [LimeSurvey webapp](https://janikvonrotz.ch/2015/04/08/install-limesurvey-webapp/)
* [Piwik for LimeSurvey](https://janikvonrotz.ch/2015/04/09/enable-piwik-for-limesurvey/)

**Koken**

* [Koken website](https://janikvonrotz.ch/2015/08/31/install-koken-website/)

## Master of Ubuntu

**Amazon Web Services**

* [s3cmd](https://janikvonrotz.ch/2014/04/10/install-s3cmd/)
* [Unattended Encrypted Incremental Backup to Amazon S3](https://janikvonrotz.ch/2014/04/25/unattended-encrypted-incremental-backup-to-amazon-s3/)

# Useful lists

Here you'll find articles about recommanded additions for your installation:

* [Backup server installations](https://janikvonrotz.ch/2014/12/08/backup-server-installations/)
* [Useful command aliases](https://janikvonrotz.ch/2014/12/08/useful-command-aliases/)

# Package sources

This is a list of documented packages and their sources. This list intends to show which sources have to be maintained to update an installation.

**automysqlbackup**
Download with wget from [Official AutoMySQLBackup SourceForge website](http://sourceforge.net/projects/automysqlbackup).

**duplicity-backup**
Clone with Git from [Official duplicity-backup GitHub repository](https://github.com/zertrin/duplicity-backup).

**HHVM**
Install with aptitude from [Official HHVM package repository](http://dl.hhvm.com/ubuntu).

**Koken**
Download with wget from [Official Koken website](http://koken.me/#dlkoken).

**LimeSurvey**
Download with wget from [Official LimeSurvey website](https://www.limesurvey.org).

**MySQL**
Install with aptitude from [Official Ubuntu package repository](http://packages.ubuntu.com/).

**Nginx**
Install with aptitude from [Official Nginx package repository](http://nginx.org/packages/ubuntu/).

**Node.js**
Download with wget from [Official Node.js website](http://nodejs.org) and compile with built-in tools.
Clone with Git from [Official Node.js GitHub repository](https://github.com/joyent/node) and compile with built-in tools.

**php5-fpm, php5 modules**
Install with aptitude from [Official Ubuntu package repository](http://packages.ubuntu.com/) and compile with built-in tools.

**phpMyAdmin**
Install with aptitude from [Official Ubuntu package repository](http://packages.ubuntu.com/).

**Piwik**
Download with wget from [Official Piwik website](ttp://builds.piwik.org).

**Piwik for LimeSurvey**
Clone with Git from [Official Piwik-for-LimeSurvey GitHub repository](https://github.com/SteveCohen/Piwik-for-Limesurvey).

**Postfix**
Install with aptitude from [Official Ubuntu package repository](http://packages.ubuntu.com/).

**QR code service**
Clone with Git from [Official QR code service GitHub repository](https://codeberg.org/janikvonrotz/QR-Generator-PHP).

**Ruby**
Install with RVM.

**RubyGem**
Install with RVM.

**RVM**
Install with script from [Official RVM website](http://rvm.io/).

**Ubuntu OS, libraries and programs**
Install with aptitude from [Official Ubuntu package repository](http://packages.ubuntu.com/).

**WordPress**
Download with wget from [Official WordPress website](http://wordpress.org).

**WPScan**
Clone with Git from [Official WPScan GitHub repository](https://github.com/wpscanteam/wpscan).