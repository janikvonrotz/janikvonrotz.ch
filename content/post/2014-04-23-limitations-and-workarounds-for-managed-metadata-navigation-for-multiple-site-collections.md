---
title: Limitations and workarounds for managed metadata navigation for multiple site collections
date: 2014-04-23T09:53:45+00:00
author: Janik von Rotz
permalink: /2014/04/23/limitations-and-workarounds-for-managed-metadata-navigation-for-multiple-site-collections/
dsq_thread_id:
  - "2632498147"
image: /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - Blog
tags:
  - collection
  - managed
  - metadata
  - mutliple
  - navigation
  - set
  - share
  - site
  - store
  - term
---
With SharePoint 2013 you can create a managed metadata navigation and share it across web applications and site collections.

But there's an ugly limitation for site collections.
<!--more-->
It seems that's not possible to share a term set navigation with more than one site collection.

When trying to do so you'll might get error messages as:

> "The selected term set is already used by another site..."

Or:

> "Error loading navigation: The Managed Navigation term set is improperly attached to the site."

However the workaround is simple and depending on the use case it could even be an improvement for a more flexible navigation.

For each site collection where you want to use a custom metadata navigation you have to create a term set in the term store.

Within these term sets you'll pin the navigation terms from the global navigation. Instead of sharing the global navigation which's not working, you're going to share specific term trees by pinning in term sets.  

In my case I've got two site collections and I'm going to add the project site to the navigation:

![Create a shared managed metadata navigation](/wp-content/uploads/2014/04/Create-a-shared-managed-metadata-navigation.gif)