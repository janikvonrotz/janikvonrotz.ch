---
title: SharePoint 2013 unable to unlock site collection
date: 2013-07-09T10:22:01+00:00
author: Janik von Rotz
slug: sharepoint-2013-unable-to-unlock-site-collection
images:
  - /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - scripting
  - SharePoint
tags:
  - powershell
  - sharepoint
---
In case you manage your SharePoint installation with PowerShell and your backup script failed, you might experience that the site collection is locked for any kind of access.
That's because SharePoint locks the site collection for backups until the backup process has finished.

And in another case like mine, it could happen that you're unable to unlock the workspace, despite your logged in as administrator.

<!--more-->

<p style="text-align: center;">![SharePoint Locked](/wp-content/uploads/2013/07/SharePoint-Locked.png)</p>

Additional to the lock state in SharePoint 2013 there's a feature to enter the site collection in a "Maintenance Mode".
This maintenance mode will lock even the administration access to the site collection.

So how can I unlock or remove the maintenance mode? It's easy just run this PowerShell script from your SharePoint/PowerShell CLI to remove the maintenance mode:

```powershell
$SPSiteAdministration =  new-object Microsoft.SharePoint.Administration.SPSiteAdministration('https://sharepoint.url.local')
$SPSiteAdministration.ClearMaintenanceMode()
```