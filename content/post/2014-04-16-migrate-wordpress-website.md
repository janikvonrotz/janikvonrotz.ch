---
id: 1979
title: Migrate WordPress website
date: 2014-04-16T07:44:47+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=1979
permalink: /2014/04/16/migrate-wordpress-website/
dsq_thread_id:
  - "2615551343"
image: /wp-content/uploads/2014/02/wordpress-logo.jpg
categories:
  - WordPress
tags:
  - altered
  - blog
  - migrate
  - new
  - obsolete
  - old
  - url
  - website
  - wordpress
---
*This post is part of my [Your own Virtual Private Server hosting solution](http://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9409150](https://gist.github.com/9409150).*  

# Introduction

This guide assumes that you're going to migrate a WordPress website from one server to an other.
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
* [phpMyAdmin website](https://janikvonrotz.ch/2014/04/14/install-phpmyadmin-website/)
* [WordPress website](https://janikvonrotz.ch/2014/04/15/install-wordpress-website/)

# Instructions

Export the WordPress `wp-content` directory from the existing installation.

Upload it to the home directory `/home/[user]/wp-content` on the new server.

Copy the `wp-content` directory to the new WordPress installation `/var/www/[wordpress]/wp-content`.

    sudo cp -r /home/[user]/wp-content/* /var/www/[wordpress]/wp-content/

Export the existing SQL WordPress database with phpMyAdmin.

Import the SQL WordPress database export into the new database with phpMyAdmin.

Update permissions for the www-data group.

    sudo chown www-data:www-data /var/www/[wordpress] -R 

## Update obsolete urls

In case you are going the change to url of the WordPress website f.g. `http://` to `https://` or `http://www.oldsiteurl.com` to `http://www.newsiteurl.com` update the WordPress database with a custom SQL statement.

Here is an example for url update using SSL only.

[code lang="sql"]
UPDATE wp_posts SET post_content = REPLACE (post_content, 'http://', 'https://');
UPDATE wp_posts SET post_content_filtered = REPLACE (post_content_filtered, 'http://', 'https://');
UPDATE wp_posts SET guid = REPLACE (guid, 'http://', 'https://');
UPDATE wp_posts SET pinged = REPLACE (pinged, 'http://', 'https://');

UPDATE wp_postmeta SET meta_value = REPLACE (meta_value, 'http://', 'https://');

UPDATE wp_options SET option_value = REPLACE (option_value, 'http://', 'https://');

UPDATE wp_comments SET comment_author_url = REPLACE (comment_author_url, 'http://', 'https://');

UPDATE wp_commentmeta SET meta_value = REPLACE (meta_value, 'http://', 'https://');
[/code]
