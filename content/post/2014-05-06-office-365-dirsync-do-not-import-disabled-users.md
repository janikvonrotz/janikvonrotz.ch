---
id: 2211
title: Office 365 DirSync do not import disabled users
date: 2014-05-06T09:13:03+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=2211
permalink: /2014/05/06/office-365-dirsync-do-not-import-disabled-users/
dsq_thread_id:
  - "2683080766"
image: /wp-content/uploads/2014/03/Office-365-flat-Logo-e1394705523286.jpg
categories:
  - Office 365
tags:
  - account
  - attribute
  - control
  - dirsync
  - disabled
  - filter
  - inactive
  - syncing
  - users
---
One of my clients mentioned that he follows people in the newsfeed who weren't employed any more.

Occasionally we disable the this kind of users in the Active Directory but don't delete them.

It seems that DirSync doesn't filter disabled accounts.
<!--more-->
To change that open the Synchronization Service Manager and navigate to > Management Agents > [your connector] > Configure Connect Filter.

Now we are going to add a new attribute filter for the account control attribute.

1. Select **user** as Data Source Object Type.
2. Click on **New**.
3. Select **userAccountControl** for Data source attribute
4. Operator is **Equal**.
5. Set value **0x202**.
6. Add the new condition and finish with OK.

<a href="https://janikvonrotz.ch/wp-content/uploads/2014/05/Configure-Connector-Filter-Account-Disabled.png"><img src="https://janikvonrotz.ch/wp-content/uploads/2014/05/Configure-Connector-Filter-Account-Disabled-1024x534.png" alt="Configure Connector Filter - Account Disabled" width="720" height="375" class="aligncenter size-large wp-image-2212" /></a>

Finally run a full sync with PowerShell.

```ps
Add-PSSnapin Coexistence-Configuration
Start-OnlineCoexistenceSync -FullSync
```

There shouldn't be any disabled users in your azure directory any more.