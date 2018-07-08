---
title: Install phpMyAdmin website
date: 2014-04-14T09:49:37+00:00
author: Janik von Rotz
slug: install-phpmyadmin-website
dsq_thread_id:
  - "2610220050"
image: /wp-content/uploads/2014/04/phpMyAdmin-Logo.jpg
categories:
  - MySQL
  - PHP
  - phpMyAdmin
tags:
  - admin
  - application
  - database
  - fpm
  - install
  - management
  - mysql
  - nginx
  - php
  - php5
  - phpmyadmin
  - server
  - sql
  - ubuntu
  - webserver
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9392925](https://gist.github.com/9392925).* 

# Introduction

phpMyAdmin is a free software tool written in PHP, intended to handle the administration of MySQL over the Web.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [Nginx](https://janikvonrotz.ch/2014/03/31/install-nginx/)
* [Nginx minimal website](https://janikvonrotz.ch/2014/04/01/nginx-minimal-website/)
* [php5-fpm](https://janikvonrotz.ch/2014/03/20/install-php5-fpm/)
* [php5-mcrypt, php5-mysql](https://janikvonrotz.ch/2014/03/25/install-php5-modules/)
* [Nginx php5-fpm website](https://janikvonrotz.ch/2014/04/11/install-nginx-php5-fpm-website/)
* [MySQL](https://janikvonrotz.ch/2014/04/07/install-mysql/)

# Installation

Start the installation phpMyAdmin.

    sudo apt-get install phpmyadmin
  
As we use nginx for this installation, hit Tab and Enter on the first prompt.

Chose <Yes> and enter the MySQL root password on the second prompt.

Create a secure password for phpMyAdmin. Don't use the MySQL root password!

Add the phpMyAdmin Nginx configuration to one of your websites.

```
server{
    ...
    
    root /usr/share;

    ...
    
    location ~ .php$ {
        ...
        
        # change the php root for phpMyAdmin
        if ($request_uri ~* /phpmyadmin) {
            set $php_root /usr/share;
        }
        
        ...
    }
}
```

Test config and reload Nginx service.

    sudo nginx -t && sudo service nginx reload
    
Open your browser on `//[host]/phpmyadmin`.
