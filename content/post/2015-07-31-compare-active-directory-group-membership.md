---
id: 3432
title: Compare Active Directory group membership
date: 2015-07-31T07:21:26+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=3432
permalink: /2015/07/31/compare-active-directory-group-membership/
dsq_thread_id:
  - "3988953744"
image: /wp-content/uploads/2013/12/PowerShell-and-ActiveDirectory.png
categories:
  - Active Directory
  - PowerShell
tags:
  - active
  - compare
  - directory
  - memebership
  - module
  - powershell
  - user
---
The following scripts allows you to compare the group membership of two users.
<!--more-->
```ps
Import-Module ActiveDirectory

$user1 = "userRef"
$user2 = "userDif"

$members1 = Get-ADPrincipalGroupMembership -Identity $user1 | Select-Object name
$members2 = Get-ADPrincipalGroupMembership -Identity $user2 | Select-Object name

$result = Compare-Object -ReferenceObject $members1 -DifferenceObject $members2 -Property name

Write-Host "`n$user1 is member of these groups in addition:" -ForegroundColor Black -BackgroundColor Yellow

$result | Where-Object{$_.SideIndicator -eq "<="} | ForEach-Object{$_.name}

Write-Host "`n$user2 is member of these goups in addition:" -ForegroundColor Black -BackgroundColor Yellow

$result | Where-Object{$_.SideIndicator -eq "=>"} | ForEach-Object{$_.name}
```

Get the latest version of this script here: [https://gist.github.com/051099188f24894502c1](https://gist.github.com/051099188f24894502c1)