---
id: 1672
title: 'Enable &#8220;Sign in as different user&#8221; option in SharePoint 2013'
date: 2014-03-24T15:49:46+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=1672
permalink: /2014/03/24/enable-sign-in-as-different-user-option-in-sharepoint-2013/
dsq_thread_id:
  - "2499366773"
image: /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - "2013"
  - log
  - sharepoint
  - sign
  - user
---
By default in SharePoint 2013 there is no option for "Sign in as different user".

To enable this option you have the update the SharePoint welcome template.<!--more-->

	C:\Program Files\Common Files\microsoft shared\Web Server Extensions\15\TEMPLATE\CONTROLTEMPLATES\Welcome.ascx

Add the following code after the `<SharePoint:MenuItemTemplate runat="server" id="ID_RequestAccess"` finisher tag `/>`.

```
<SharePoint:MenuItemTemplate runat="server" ID="ID_LoginAsDifferentUser"
  Text="<%$Resources:wss,personalactions_loginasdifferentuser%>"
  Description="<%$Resources:wss,personalactions_loginasdifferentuserdescription%>"
  MenuGroupId="100"
  Sequence="100"
  UseShortId="true"
/>
```

# Source

[How to enable "Sign in as different user" option in SharePoint 2013](http://www.codeproject.com/Tips/684751/How-to-enable-Sign-in-as-different-user-option-in)