---
id: 531
title: Find dead SharePoint ActiveDirectory Groups
date: 2013-09-25T15:33:07+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=531
permalink: /2013/09/25/find-dead-sharepoint-activedirectory-groups/
dsq_thread_id:
  - "1795902964"
image: /wp-content/uploads/2013/07/PowerShell.png
categories:
  - Active Directory
  - PowerShell
  - SharePoint
tags:
  - access
  - activedirectory
  - dead
  - groups
  - handle
  - issue
  - managment
  - rights
  - sharepoint
---
The are three ways to handle access rights in SharePoint.

<ul>
    <li>Using ActiveDirectory Groups</li>
    <li>Using SharePoint Groups</li>
    <li>Using both of them</li>
</ul>

I personally recommend to use the first suggestion. Managing the access rights in one system is much easier to administrate, no switching or log off for administration work.

In our SharePoint installation I create for each securable resource and rights type a ActiveDirectory group and assign them organization groups.

A huge disadvantage of this strategy is that after a period of adding ActiveDirectory groups it's hard to know which of those groups are really required.

<!--more-->

I could handle this issue with a simple script which compares all SharePoint ActiveDirectory groups and the All ActiveDirectory groups from a specific OU against.

```powershell
Import-Module ActiveDirectory

$Domain = "$((Get-ADDomain).Name)"

$ADGroups = Get-ADGroup -Filter "*" -SearchBase "OU=SharePoint,OU=Services,OU=vblusers2,DC=vbl,DC=ch"

$SPGroups = (
    Get-SPWebs | %{
        if($_.HasUniqueRoleAssignments){
            $Url = $_.Url
            $_.RoleAssignments | Where{$_.Member.IsDomainGroup} | %{ $_ | Select-Object @{Name = "Member"; Expression = {$_.member -replace ($Domain + "\"),""}}, @{Name = "Url"; Expression = {$Url}},@{Name = "Type"; Expression = {"Website"}}}
        }
    }
    )+(

    Get-SPLists | %{
        if($_.HasUniqueRoleAssignments){
            $Url = ([uri]$_.Parentweb.Url).Scheme + "://" + ([uri]$_.Parentweb.Url).host + $_.DefaultViewUrl
            $_.RoleAssignments | Where{$_.Member.IsDomainGroup} | %{ $_ | Select-Object @{Name = "Member"; Expression = {$_.member -replace ($Domain + "\"),""}}, @{Name = "Url"; Expression = {$Url}},@{Name = "Type"; Expression = {"List"}}}
        }
    }
)

$ADGroups | where{ -not (($SPGroups | select Member) -match $_.Name)} | select name
```

<a href="https://gist.github.com/6699783">https://gist.github.com/6699783</a>

<h1>Requirements</h1>

<ul>
    <li><a href="https://github.com/janikvonrotz/Powershell-Profile">Powershell-Profile</a></li>
</ul>