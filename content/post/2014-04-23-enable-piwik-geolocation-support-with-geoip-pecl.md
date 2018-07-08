---
title: Enable Piwik geolocation support with GeoIP PECL
date: 2014-04-23T08:57:31+00:00
author: Janik von Rotz
permalink: /2014/04/23/enable-piwik-geolocation-support-with-geoip-pecl/
dsq_thread_id:
  - "2632437056"
image: /wp-content/uploads/2014/12/logo-piwik-e1418200408662.png
categories:
  - Piwik
tags:
  - analytics
  - city
  - country
  - enable
  - geo
  - geoip
  - ip
  - location
  - pecl
  - piwik
  - support
  - tagging
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9758234](https://gist.github.com/9758234).*  

# Introduction

By default Piwik uses the provider location to guess a visitor's country based on the language they use. This is not very accurate, so they recommend installing and using GeoIP.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [libgeoip-dev](https://janikvonrotz.ch/2014/03/25/install-ubuntu-development-libraries/)
* [Nginx](https://janikvonrotz.ch/2014/03/31/install-nginx/)
* [Nginx minimal website](https://janikvonrotz.ch/2014/04/01/nginx-minimal-website/)
* [php5-fpm](https://janikvonrotz.ch/2014/03/20/install-php5-fpm/)
* [php5-dev, php5-geoip, php5-mysql](https://janikvonrotz.ch/2014/03/25/install-php5-modules/)
* [Nginx php5-fpm website](https://janikvonrotz.ch/2014/04/11/install-nginx-php5-fpm-website/)
* [MySQL](https://janikvonrotz.ch/2014/04/07/install-mysql/)
* [Piwik website](https://janikvonrotz.ch/2014/04/22/install-piwik-website/)

# Installation

Download the latest GeoLite database

    sudo wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz -P /var/www/[piwik]/misc/

Unzip the database.

    sudo gunzip /var/www/[piwik]/misc/GeoLiteCity.dat.gz

Rename the file.

    cd /var/www/[piwik]/misc
    sudo mv GeoLiteCity.dat GeoIPCity.dat

Update the access rights.

    sudo chown www-data:www-data GeoIPCity.dat

Update the php configuration file.

    sudo vi /etc/php5/fpm/php.ini

Add the geoip configuration.
    
    geoip.custom_directory = /var/www/[piwik]/misc

Restart Nginx and php5-fpm service.

    sudo nginx -t && sudo service nginx reload
    sudo service php5-fpm restart

Now open your piwik installation on `//[host]/piwik/index.php?module=UserCountry&action=adminIndex` and check the `GeoIP (PECL)` option.

In addition the the GeoLite download url to the `Download URL` field and click save.

# Source

[How do I install the GeoIP Geo location PECL extension?](http://piwik.org/faq/how-to/#faq_164)