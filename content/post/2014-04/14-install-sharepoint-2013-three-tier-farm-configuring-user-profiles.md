---
title: 'Install SharePoint 2013 Three-tier Farm - Configuring User Profiles'
date: 2014-04-14T09:24:12+00:00
author: Janik Vonrotz
slug: install-sharepoint-2013-three-tier-farm-configuring-user-profiles
images:
  - /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - sharepoint
---
*This post of is part of my [Install SharePoint 2013 Three-tier Farm](https://janikvonrotz.ch/projects/install-sharepoint-2013-three-tier-farm/) project.*

# Filter settings

By default SharePoint won't user profiles of disabled accounts. To change that update the connection filter on the User Profile Sync service.
<!--more-->
![User Profile sync connection filter](/wp-content/uploads/2014/04/User-Profile-sync-connection-filter.png)

As you can see I've added a rule for the userAccountControl. Only activated user accounts get a MySite profile.

# Enable page lock

This is another SQL database best practice to enable page locks on the User Profile Sync database indexes.

First run this SQL statement:

```sql
USE "SP_AS_UPS_Sync"
GO
SELECT 'ALTER INDEX ' + I.Name + ' ON ' + T.Name + ' SET (ALLOW_PAGE_LOCKS = ON)' As Command
FROM sys.indexes I
LEFT OUTER JOIN sys.tables T ON I.object_id = t.object_id
WHERE I.allow_page_locks = 0 AND T.name IS NOT NULL
```

This statement will output the alter statement foreach index. Run this output as well.

```sql
ALTER INDEX IX_RequestOutput_ObjectKey ON RequestOutput SET (ALLOW_PAGE_LOCKS = ON)
ALTER INDEX IX_RequestOutput_RequestIdentifier_ObjectID_AttributeKey_ValueReference ON RequestOutput SET (ALLOW_PAGE_LOCKS = ON)
...
```