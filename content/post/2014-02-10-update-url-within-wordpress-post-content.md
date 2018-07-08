---
title: Update Url within WordPress Post Content
date: 2014-02-10T13:06:55+00:00
author: Janik von Rotz
permalink: /2014/02/10/update-url-within-wordpress-post-content/
dsq_thread_id:
  - "2246601355"
image: /wp-content/uploads/2014/02/wordpress-logo.jpg
categories:
  - WordPress
tags:
  - changed
  - content
  - domain
  - how
  - name
  - obsolete
  - queries
  - query
  - replace
  - sql
  - update
  - urls
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