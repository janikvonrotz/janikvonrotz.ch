---
title: 'Install SharePoint 2013 Three-tier Farm - Migrate SharePoint 2010 Data'
date: 2014-04-14T09:36:40+00:00
author: Janik Vonrotz
slug: install-sharepoint-2013-three-tier-farm-migrate-sharepoint-2010-data
images:
  - /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - migration
  - sharepoint
  - update
  - upgrade
---
*This post of is part of my [Install SharePoint 2013 Three-tier Farm](https://janikvonrotz.ch/projects/install-sharepoint-2013-three-tier-farm/) project.*

This time I'll show you my approach for a SharePoint 2010 to 2013 migration.
<!--more-->
# Preparation

* Remove third party solutions as they might won't be necessary or compatible in SharePoint 2013.
* Report your existing DB security (a migration will remove existing DB login permission roles).

# Migration

Migrate content databases in their specific dependency order. For example if you haven an site collection with site columns that make use of the Managed Metadata service you have to migrate the Managed Metadata service database first.

# Migrate Managed Metadata service

1. Backup the existing database
2. Remove the service application in SharePoint 2013
3. Copy the existing database export to the SharePoint 2013 server
4. Restore the database
5. Rename the database according to the new naming concept (only necessary if the it has changed)
6. Update the permissions of the database according to the permission report
7. Create a new Managed Metadata service application and select the imported database
8. Update the term store administrators

# Migrate site collection

1. Backup the existing content database
2. Detach the site collection database in SharePoint 2013
3. Delete the detached content database in SharePoint 2013
4. Copy the existing database export to the SharePoint 2013 server
5. Restore the database
6. Rename the database according to the new naming concept (only necessary if the it has changed)
7. Update the permissions of the database according to the permission report
8. Test the restored database with the SharePoint PowerShell command line `Test-SPContentDatabase -Name DatabaseName -WebApplication http://example.sharepoint.com`
9. This test will output some compatibility issues, take action according to the error messages
10. Attach the database to the farm with PowerShell `Mount-SPContentDatabase -Name DatabaseName -DatabaseServer SQLServer -WebApplication http://example.sharepoint.com -AssignNewDatabaseId` 
11. Upate site collection administrators on the site collection
12. Disable locks of the collection
13. Delete Office Web Apps site collection
14. Run `Get-SPsite` and look after the CompatibilityLevel attribute. Everything less than 15 needs an visual upgrade
15. Update the visual design for all SharePoint site `Get-SPSite http://example.sharepoint.com | Upgrade-SPSite -VersionUpgrade`
16. Update to username to claims by entering these commands in PowerShell:
```powershell
$SPWebApplication = Get-SPWebApplication http://sharepoint.vbl.ch
$SPWebApplication.MigrateUsers($true)
$SPWebApplication.ProvisionGlobally()
```
