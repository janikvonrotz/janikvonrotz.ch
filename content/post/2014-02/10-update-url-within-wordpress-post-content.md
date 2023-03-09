---
title: Update Url within WordPress Post Content
date: 2014-02-10T13:06:55+00:00
author: Janik von Rotz
slug: update-url-within-wordpress-post-content
images:
  - /wp-content/uploads/2014/02/wordpress-logo.jpg
categories:
  - Web development
tags:
  - sql server
  - wordpress
---
Recently I've change the url for one of my GitHub projects.

<ul>
    <li>Old: https://github.com/janikvonrotz/PowerShell-<strong>Profile</strong></li>
    <li>New: https://github.com/janikvonrotz/PowerShell-<strong>PowerUp</strong></li>
</ul>

As you might know, I've used this url in several posts.
<span style="line-height: 1.5;">In order to make this obsolete links I have to replace them in every posts.</span>

<!--more-->

This easy done with the following SQL query:

```sql

UPDATE wp_posts SET post_content = REPLACE (post_content, 'https://www.oldsiteurl.com', 'https://www.newsiteurl.com')

```

To run this SQL script you have to able to access the WordPress MySQL database.

To get you further into the possibilities of manipulating the WordPress database content with queries, I've collected the most useful queries in a gist: <a href="https://gist.github.com/8914414">https://gist.github.com/8914414</a>