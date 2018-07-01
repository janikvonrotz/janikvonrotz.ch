---
id: 1894
title: 'Install SharePoint 2013 Three-tier Farm - Configuring User Profiles'
date: 2014-04-14T09:24:12+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=1894
permalink: /2014/04/14/install-sharepoint-2013-three-tier-farm-configuring-user-profiles/
dsq_thread_id:
  - "2610173991"
image: /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - connection
  - filter
  - index
  - profile
  - settings
  - sharepoint
  - sql
  - user
---
*This post of is part of my [Install SharePoint 2013 Three-tier Farm](https://janikvonrotz.ch/projects/install-sharepoint-2013-three-tier-farm/) project.*

# Filter settings

By default SharePoint won't user profiles of disabled accounts. To change that update the connection filter on the User Profile Sync service.
<!--more-->
<img src="https://janikvonrotz.ch/wp-content/uploads/2014/04/User-Profile-sync-connection-filter.png" alt="User Profile sync connection filter" width="940" height="209" class="aligncenter size-full wp-image-1895" />

As you can see I've added a rule for the userAccountControl. Only activated user accounts get a MySite profile.

# Enable page lock

This is another SQL database best practice to enable page locks on the User Profile Sync database indexes.

First run this SQL statement:

[code lang="sql"]
USE "SP_AS_UPS_Sync"
GO
SELECT 'ALTER INDEX ' + I.Name + ' ON ' + T.Name + ' SET (ALLOW_PAGE_LOCKS = ON)' As Command
FROM sys.indexes I
LEFT OUTER JOIN sys.tables T ON I.object_id = t.object_id
WHERE I.allow_page_locks = 0 AND T.name IS NOT NULL
[/code]

This statement will output the alter statement foreach index. Run this output as well.

[code lang="sql"]
ALTER INDEX IX_RequestOutput_ObjectKey ON RequestOutput SET (ALLOW_PAGE_LOCKS = ON)
ALTER INDEX IX_RequestOutput_RequestIdentifier_ObjectID_AttributeKey_ValueReference ON RequestOutput SET (ALLOW_PAGE_LOCKS = ON)
...
[/code]