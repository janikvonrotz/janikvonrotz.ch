---
title: SharePoint 2013 navigation sub menu titles cut off
date: 2014-03-10T14:35:34+00:00
author: Janik Vonrotz
slug: sharepoint-2013-navigation-sub-menu-titles-cut-off
images:
  - /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - sharepoint
  - sub titles
---
By default SharePoint 2013 cuts off long sub menu titles in the navigation.

[![SharePoint 2013 Menu without space](/wp-content/uploads/2014/03/SharePoint-2013-Menu-without-space.jpg)](/wp-content/uploads/2014/03/SharePoint-2013-Menu-without-space.jpg)

SharePoint expects only titles without spaces in between words.
This is miss behaviour is simply solved by adding a custom css in the master page template.
<!--more-->
[![SharePoint 2013 Add CSS to Master Page](/wp-content/uploads/2014/03/SharePoint-2013-Add-CSS-to-Master-Page-1024x608.jpg)](/wp-content/uploads/2014/03/SharePoint-2013-Add-CSS-to-Master-Page.jpg)

```css
ul.dynamic li {
    white-space: nowrap;
}
```

In the end it should look somehow like this.

[![SharePoint 2013 Menu with space](/wp-content/uploads/2014/03/SharePoint-2013-Menu-with-space.jpg)](/wp-content/uploads/2014/03/SharePoint-2013-Menu-with-space.jpg)