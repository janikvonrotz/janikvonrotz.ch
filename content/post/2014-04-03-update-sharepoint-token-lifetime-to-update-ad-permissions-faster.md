---
title: Update SharePoint Token Lifetime to update AD permissions faster
date: 2014-04-03T09:01:03+00:00
author: Janik von Rotz
slug: update-sharepoint-token-lifetime-to-update-ad-permissions-faster
dsq_thread_id:
  - "2582712894"
image: /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - active
  - ad
  - changes
  - claims
  - directory
  - effect
  - group
  - immediately
  - life
  - long
  - sharepoint
  - time
  - token
  - update
---
Since SharePoint 2013 only supports claim based authentication I discovered that updates in SharePoint Active Directory groups do not take effect immediately.

Thanks to [Ryan McIntyre](http://blog.randomdust.com/index.php/2013/06/sharepoint-2013-claim-expiration-and-ad-sync/) there's a simple fix for that issue.

By adjusting the lifetime of the claims token you can shorten the time it takes to update the Active Directory group changes.
<!--more-->
```powershell
if(-not (Get-PSSnapin "Microsoft.SharePoint.PowerShell" -ErrorAction SilentlyContinue)){Add-PSSnapin "Microsoft.SharePoint.PowerShell"}

# update SharePoint cache token lifetime

$SPContentService = [Microsoft.SharePoint.Administration.SPWebService]::ContentService
$SPContentService.TokenTimeout = (New-TimeSpan -minutes 5)
$SPContentService.Update()

# udpate SharePoint claims token lifetime

$SPSecurityTokenServiceConfig = Get-SPSecurityTokenServiceConfig
$SPSecurityTokenServiceConfig.WindowsTokenLifetime = (New-TimeSpan –minutes 5)
$SPSecurityTokenServiceConfig.FormsTokenLifetime = (New-TimeSpan -minutes 5)

# if you happen to set a lifetime that is shorter than the expiration window user will be blocked from accessing the site.
$SPSecurityTokenServiceConfig.LogonTokenCacheExpirationWindow = (New-TimeSpan -minutes 4)
$SPSecurityTokenServiceConfig.Update()
```

Get the latest version of this code snippet here: [https://gist.github.com/9950021](https://gist.github.com/9950021)