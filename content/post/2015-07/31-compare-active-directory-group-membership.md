---
title: Compare Active Directory group membership
date: 2015-07-31T07:21:26+00:00
author: Janik Vonrotz
slug: compare-active-directory-group-membership
images:
  - /wp-content/uploads/2013/12/PowerShell-and-ActiveDirectory.png
categories:
  - scripting
tags:
  - membership
  - powershell
---
The following scripts allows you to compare the group membership of two users.
<!--more-->
```powershell
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