---
title: Update SharePoint ActiveDirectory Group Displayname
date: 2013-12-16T17:00:39+00:00
author: Janik von Rotz
slug: update-sharepoint-activedirectory-group-displayname
dsq_thread_id:
  - "2054243559"
image: /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - activedirectory
  - change
  - display
  - groups
  - name
  - sharepoint
  - wrong
---
When the name of an Active Directory group has been changed, this change won't affect the display name showed in the SharePoint permission editor.

To update the the name for all Active Directory groups you can run this snippet on the SharePoint server:

<!--more-->

```powershell

if ((Get-PSSnapin 'Microsoft.SharePoint.PowerShell' -ErrorAction SilentlyContinue) -eq $null){Add-PSSnapin 'Microsoft.SharePoint.PowerShell'}

$SPSiteFilter = "https://sharepoint.domain.ch"

Get-SPSite | where{$SPSiteFilter -contains $_.Url} | %{

    $_.rootweb.siteusers | where{($_.DisplayName -ne $_.UserLogin) -and $_.IsDomainGroup} | %{

        Write-Host "Change: $($_.DisplayName) to: $($_.UserLogin)"

        $_.DisplayName = $_.UserLogin
        $_.Name = $_.UserLogin
        $_.Update()

    }
}
```

This snippet is part of my SharePoint default settings script: <a href="https://gist.github.com/7871902">https://gist.github.com/7871902</a>