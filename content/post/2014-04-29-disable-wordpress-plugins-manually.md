---
id: 2131
title: Disable WordPress plugins manually
date: 2014-04-29T08:05:06+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=2131
permalink: /2014/04/29/disable-wordpress-plugins-manually/
dsq_thread_id:
  - "2648145178"
image: /wp-content/uploads/2014/02/wordpress-logo.jpg
categories:
  - WordPress
tags:
  - disable
  - fix
  - malicious
  - manually
  - plugin
  - update
  - wordpress
---
With the WordPress 3.9 Update I also received several plugin updates for my blogs.

One Update was for WP-Piwik. As I've updated the plugin I got several errors and the site become unreachable.

So the only way the to disable this malicious plugin was either via ftp or with phpMyAdmin.

I've decided to do it with phpMyAdmin.
<!--more-->
The settings for the plugin activation are stored in the `wp_options` table.

To get the data for the activated plugins run this sql statement:

```sql
SELECT * FROM wp_options WHERE option_name = 'active_plugins';
```

By editing the option_value field you'll get a semicolon separated list of plugins and by simply removing a value you'll deactivate the plugin.

For WP-Piwik it was:

```sql
i:8;s:21:"wp-piwik/wp-piwik.php";
```