---
title: Install piwik website
date: 2014-04-22T06:43:16+00:00
author: Janik von Rotz
slug: install-piwik-website
dsq_thread_id:
  - "2629575470"
image: /wp-content/uploads/2014/12/logo-piwik-e1418200408662.png
categories:
  - Piwik
tags:
  - analytics
  - database
  - install
  - mysql
  - nginx
  - piwik
  - server
  - ubuntu
  - web
  - website
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9640540](https://gist.github.com/9640540).*  

# Introduction

Piwik is the leading open source web analytics platform that gives you valuable insights into your website’s visitors, your marketing campaigns and much more, so you can optimize your strategy and online experience of your visitors.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [Nginx](https://janikvonrotz.ch/2014/03/31/install-nginx/)
* [Nginx minimal website](https://janikvonrotz.ch/2014/04/01/nginx-minimal-website/)
* [php5-fpm](https://janikvonrotz.ch/2014/03/20/install-php5-fpm/)
* [php5-mysql](https://janikvonrotz.ch/2014/03/25/install-php5-modules/)
* [MySQL](https://janikvonrotz.ch/2014/04/07/install-mysql/)
* [Nginx php5-fpm website](https://janikvonrotz.ch/2014/04/11/install-nginx-php5-fpm-website/)

# Installation

Create the new Piwik website folder.

    sudo mkdir /var/www/[piwik]
    cd /var/www/[piwik]

Download latest piwik

    sudo wget http://builds.piwik.org/latest.tar.gz
    sudo tar -xzvf latest.tar.gz

Copy Piwik content and delete unnecessary files.
    
    sudo cp -r ./piwik/* ./
    sudo rm -r piwik
    sudo rm latest.tar.gz "How to install Piwik.html"
  
Let's create the MySQL Piwik database and user.

    mysql -u root -p
    
Enter the MySQL root user password.

Create the Piwik database.

    CREATE DATABASE [piwik];
    
Create the Piwik database user.

    CREATE USER [piwik]@localhost;

Set the password for the Piwik database user.

    SET PASSWORD FOR [piwik]@localhost = PASSWORD("[password]");
    
Grant Piwik user full access on Piwik database.

    GRANT ALL PRIVILEGES ON [piwik].* TO [piwik]@localhost IDENTIFIED BY '[password]';
    
Refresh MySQL and exit.

    FLUSH PRIVILEGES;
    exit
    
Add the Nginx configuration to an existing website.

```
server{
    
    ...
    
    location /piwik{
        root /var/www;
    }
    
    ...
    
    location ~ .php$ {
        
        ...
        
        if ($request_uri ~* /piwik) {
            set $php_root /var/www;
        }
        
        ...
    }
}
```

Provide access to the piwik folder.

    sudo chown -R www-data:www-data /var/www/piwik

Finally let's add the archive cron job which will highly improve the processing time for your piwik reports.

Add a new cron job.

    sudo vi /etc/cron.d/piwik-archive

Add this content to the cron file.

    MAILTO="[mail@example.com]"
    5 * * * * www-data /usr/bin/php5 /var/www/[piwik]/console core:archive --url=http://[host]/piwik/ > /var/log/piwik/archive.log

Then create the log folder and grant access for the user.

    sudo mkdir /var/log/piwik
    sudo chown www-data:www-data piwik

Test config and reload Nginx service.

    sudo nginx -t && sudo service nginx reload
    
Open your browser on `//[host]/piwik` and install the Piwik website.

# Source

[How to Install Piwik on Ubuntu by AdminEmpire](http://www.adminempire.com/how-to-install-piwik-on-ubuntu/)  