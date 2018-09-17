---
title: Disable Trackbacks and Pingbacks in WordPress
date: 2014-02-27T08:24:50+00:00
author: Janik von Rotz
slug: disable-trackbacks-and-pingbacks-in-wordpress
dsq_thread_id:
  - "2328729110"
images:
  - /wp-content/uploads/2014/02/wordpress-logo.jpg
categories:
  - WordPress
tags:
  - pingbacks
  - solutions
  - trackbacks
  - wordpress
---
Trackbacks and Pingbacks can be abused for spam and even <a href="https://krebsonsecurity.com/2014/03/blogs-of-war-dont-be-cannon-fodder/" title="DDOS attacks">DDOS attacks</a>.

<header>To disable trackbacks and pingbacks completely on WordPress, do the following:

</header>

<div>
<h1>Disable for new posts</h1>
Uncheck the following setting:

Settings>Discussion>Allow link notifications from other blogs (pingbacks and trackbacks)

<!--more-->

<strong>This will disable trackbacks and pingbacks for all future posts, but not for existing posts.</strong>
<h1>Disable them on existing posts</h1>
<ul>
    <li>Go to your Posts page</li>
    <li>Click "Screen Options" in the top right</li>
    <li>View "x" number of posts value to something reasonable, based on how many posts you have</li>
    <li>Click the "Select All" box to select all posts visible on the page</li>
    <li>Click "Edit" from the Bulk Actions dropdown menu</li>
    <li>Set the "Pings" dropdown to be "Do not allow"</li>
</ul>
This should <strong>disable them for all existing posts</strong>. Last step, pages.
<h1>Disable them on existing pages</h1>

Unfortunately I couldn’t find a way to do this in bulk, or via the "Quick Edit" screen, so you'll have to go through each page and manually uncheck the option under the "Discussion" tab.

Or if you're able to access the MySQL server with phpMyAdmin or a MySQL CLI you can run this SQL statement.

```sql
-- Globally disable pingbacks/trackbacks for all users
UPDATE wp_posts SET ping_status = 'closed'
```

Don't be confused by the column name `wp_posts` it also contains pages.

<strong>After completing these 3 steps, you should have no more trackbacks or pingbacks on your WordPress site.</strong>

</div>