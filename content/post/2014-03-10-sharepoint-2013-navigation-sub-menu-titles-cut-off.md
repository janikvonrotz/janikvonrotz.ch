---
id: 1478
title: SharePoint 2013 navigation sub menu titles cut off
date: 2014-03-10T14:35:34+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=1478
permalink: /2014/03/10/sharepoint-2013-navigation-sub-menu-titles-cut-off/
dsq_thread_id:
  - "2400906975"
image: /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - "2013"
  - css
  - cut
  - fix
  - issue
  - 'off'
  - problem
  - sharepoint
  - solved
  - sub
  - titles
  - ugly
  - wtf
---
By default SharePoint 2013 cuts off long sub menu titles in the navigation.

<a href="https://janikvonrotz.ch/wp-content/uploads/2014/03/SharePoint-2013-Menu-without-space.jpg"><img src="https://janikvonrotz.ch/wp-content/uploads/2014/03/SharePoint-2013-Menu-without-space.jpg" alt="SharePoint 2013 Menu without space" width="141" height="237" class="size-full wp-image-1481" /></a>

SharePoint expects only titles without spaces in between words.
This is miss behaviour is simply solved by adding a custom css in the master page template.
<!--more-->
<a href="https://janikvonrotz.ch/wp-content/uploads/2014/03/SharePoint-2013-Add-CSS-to-Master-Page.jpg"><img src="https://janikvonrotz.ch/wp-content/uploads/2014/03/SharePoint-2013-Add-CSS-to-Master-Page-1024x608.jpg" alt="SharePoint 2013 Add CSS to Master Page" width="620" height="368" class="size-large wp-image-1479" /></a>

[code lang=css]
ul.dynamic li {
    white-space: nowrap;
}
```

In the end it should look somehow like this.

<a href="https://janikvonrotz.ch/wp-content/uploads/2014/03/SharePoint-2013-Menu-with-space.jpg"><img src="https://janikvonrotz.ch/wp-content/uploads/2014/03/SharePoint-2013-Menu-with-space.jpg" alt="SharePoint 2013 Menu with space" width="251" height="155" class="size-full wp-image-1480" /></a>