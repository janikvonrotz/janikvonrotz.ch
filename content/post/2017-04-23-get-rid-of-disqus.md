---
title: Get rid of Disqus
date: 2017-04-23T08:44:28+00:00
author: Janik von Rotz
permalink: /2017/04/23/get-rid-of-disqus/
categories:
  - Blog
  - WordPress
tags:
  - alternative
  - comment
  - disable
  - disqus
  - load
  - network
  - page
  - performance
  - privacy
  - system
  - time
  - tracking
---
Recently I read an article on HN (Hacker News) [Replacing Disqus with Github Comments](http://donw.io/post/github-comments/) and decided to drop the Disqus commenting system on this site. A long time ago I've added Disqus to my page because of it was easy to use and had out-of-the-box spam prevention. The cost of it as I see now was a heavy breach in privacy and site performance. When I read the article mentioned above I was shocked what I was forcing on people visiting my site. Just to give you an idea what's going on:

<!--more-->

 * Page load time - Disqus is everything else than a slick commenting system. It uses way too much of the users resources to load some comments on a page.
 * User tracking - Disqus is connected to many advertising networks and tracking services. If you're using Disqus on your site it will track every movement of your readers.
 * Product of Disqus - Disqus is free to use, in result your site becomes the product. I don't want to be part of this malicious network.

TL;TR: Disqus is a malicious network which invades users privacy using the resources of visitors. Nothing is free.

If you're using WordPress to host your site here are some alternatives to Disqus:

* [Isso](https://posativ.org/isso/) - A commenting system very similar to Disqus but not as invasive.
* Default commenting system - Why not use the default WordPress commenting system in conjunction with the Jetpack plugin.
* [http://www.discourse.org/](Discourse) - Despite the similar name Discourse delivers a different commenting system for the open source community.

I've decided to use the default commenting system, luckily WordPress or the Disqus plugin stored the comments in the database and I only had to disable the Disqus plugin to switch to default comments.

