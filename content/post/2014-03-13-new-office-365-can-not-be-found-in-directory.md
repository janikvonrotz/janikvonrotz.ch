---
title: New Office 365 user can not be found in directory
date: 2014-03-13T11:22:32+00:00
author: Janik von Rotz
permalink: /2014/03/13/new-office-365-can-not-be-found-in-directory/
dsq_thread_id:
  - "2420379308"
image: /wp-content/uploads/2014/03/Office-365-flat-Logo-e1394705523286.jpg
categories:
  - Office 365
tags:
  - "365"
  - found
  - impossible
  - in directory
  - invalid
  - login
  - new
  - office
  - online
  - provisioning
  - user
---
As part of the user provisioning process in my company every user account gets an Office 365 license.

2 days ago in the after noon I've created an new user account, synced it with DirSync and applied an new Office 365 E1 license.

When I've tried access the SharePoint Online website via ADFS SSO, I received an error message like this:
<!--more-->

```text
user@example.com can not be found in directory 

Correlation ID: 812b329c-0ad4-702d-d866-1a5dd6a5d715
Date and Time: 13/03/2014 08:58:17
URL: https://example.vbluzern.com/?wa=wsignin1.0 
User: user@example.com
Issue Type: Partner User Invalid
```

I tried everything to get rid of this error. Recreating the user account, disable, enable, assign another license, compare with working accounts, but nothing could solve this issue.

However as I've browsed in the Technet forums I came along this issue <a href="https://community.office365.com/en-us/forums/154/p/178626/526777.aspx">Partner User Invalid when creating a new user account.</a>. They described almost the same issue as mine. In the last answer somebody proposed simply to wait 24 hours after assigning the license.

And guess what! It worked!

24 hours after I've assigned the Office 365 license I could login with the new user account.

Thank you Microsoft for these detailed error reports :)