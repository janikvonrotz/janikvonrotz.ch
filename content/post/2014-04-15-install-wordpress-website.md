---
title: Install WordPress website
date: 2014-04-15T07:00:32+00:00
author: Janik von Rotz
slug: install-wordpress-website
dsq_thread_id:
  - "2612708517"
image: /wp-content/uploads/2014/02/wordpress-logo.jpg
categories:
  - WordPress
tags:
  - blog
  - easy
  - install
  - ubuntu
  - website
  - wordpress
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9408580](https://gist.github.com/9408580).*  

# Introduction

WordPress is web software you can use to create a beautiful website or blog.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [Nginx](https://janikvonrotz.ch/2014/03/31/install-nginx/)
* [Nginx minimal website](https://janikvonrotz.ch/2014/04/01/nginx-minimal-website/)
* [php5-fpm](https://janikvonrotz.ch/2014/03/20/install-php5-fpm/)
* [php5-mysql](https://janikvonrotz.ch/2014/03/25/install-php5-modules/)
* [Nginx php5-fpm website](https://janikvonrotz.ch/2014/04/11/install-nginx-php5-fpm-website/)
* [MySQL](https://janikvonrotz.ch/2014/04/07/install-mysql/)
* [Increased Max Upload for php5-fpm website](https://janikvonrotz.ch/2014/04/11/increase-max-upload-for-php5-fpm-website/)

# Installation

Create the website directory

    sudo mkdir /var/www/[wordpress]/

Open the WordPress site directory

    cd /var/www/[wordpress]/

Download latest WordPress package and untar it

    sudo wget http://wordpress.org/latest.tar.gz
    sudo tar -xzvf latest.tar.gz
    
Copy the untared files to the current folder and delete the other files
    
    sudo cp -r ./wordpress/* ./
    sudo rm -r wordpress
    sudo rm latest.tar.gz

Let's create the MySQL WordPress database and user.

    mysql -u root -p
    
Enter the MySQL root user password.

Create the WordPress database.

    CREATE DATABASE [wordpress];
    
Create the WordPress database user.

    CREATE USER [wordpress]@localhost;

Set the password for the WordPress database user.

    SET PASSWORD FOR [wordpress]@localhost = PASSWORD("[password]");
    
Grant WordPress user full access on WordPress database.

    GRANT ALL PRIVILEGES ON [wordpress].* TO [wordpress]@localhost IDENTIFIED BY '[password]';
    
Refresh MySQL and exit.

    FLUSH PRIVILEGES;
    exit

Add the Nginx configuration to the WordPress website.
```
server{    
  ...

  location / {
    try_files $uri $uri/ /index.php?$args;
  }
 
  ...
}
```
Update permissions for the `www-data` group.

    sudo chown www-data:www-data /var/www/[wordpress] -R 
    
Test config and reload Nginx service.

    sudo nginx -t && sudo service nginx reload

Open the browser again on `//[host]` and install the WordPress blog.

# Source

[WordPress Nginx Codex](http://codex.wordpress.org/Nginx)
[How To Install Wordpress with nginx on Ubuntu 12.04 by Digital Ocean](https://www.digitalocean.com/community/articles/how-to-install-wordpress-with-nginx-on-ubuntu-12-04)