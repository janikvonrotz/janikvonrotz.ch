---
id: 3126
title: Create WordPress admin user with sql only
date: 2015-03-24T17:54:28+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=3126
permalink: /2015/03/24/create-wordpress-admin-user-with-sql-only/
dsq_thread_id:
  - "3642482614"
image: /wp-content/uploads/2014/02/wordpress-logo.jpg
categories:
  - WordPress
tags:
  - add
  - admin
  - reset
  - script
  - snippet
  - sql
  - user
  - wordpress
---
If you don't have access to you WordPress website anymore, but still can update the MySQL database. You can easily add a new admin user with this sql script:

[code lang="sql"]
USE &lt;database&gt;;
 
SET @username = '&lt;username&gt;';
SET @password = MD5('&lt;password&gt;');
SET @fullname = '&lt;Firstname Lastname&gt;';
SET @email = '&lt;login@example.com&gt;';
SET @url = '&lt;http://example.com/&gt;';
 
INSERT INTO `wp_users` (`user_login`, `user_pass`, `user_nicename`, `user_email`, `user_url`, `user_registered`, `user_status`, `display_name`) VALUES (@username, @password, @fullname, @email, @url, NOW(), '0', @fullname);
 
SET @userid = LAST_INSERT_ID();
INSERT INTO `wp_usermeta` (`user_id`, `meta_key`, `meta_value`) VALUES (@userid, 'wp_capabilities', 'a:1:{s:13:&quot;administrator&quot;;s:1:&quot;1&quot;;}');
INSERT INTO `wp_usermeta` (`user_id`, `meta_key`, `meta_value`) VALUES (@userid, 'wp_user_level', '10');
[/code]

Get the latest version of this snippet here: [https://gist.github.com/25d9f243fc20532cb1b5](https://gist.github.com/25d9f243fc20532cb1b5)