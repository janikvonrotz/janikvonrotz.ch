---
id: 824
title: Change Active Directory User Password Expiration Mode
date: 2013-12-11T17:35:45+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=824
permalink: /2013/12/11/change-active-directory-user-password-expiration-mode/
dsq_thread_id:
  - "2043937580"
image: /wp-content/uploads/2013/12/PowerShell-and-ActiveDirectory.png
categories:
  - Active Directory
tags:
  - activedirectory
  - change
  - expiration
  - mode
  - password
  - powershell
  - snippet
---
To change an Active Directory users password expiration mode you can use this PowerShell snippet:

```powershell
Import-Module ActiveDirectory

Get-ADGroupMember "Group1" -Recursive |
Get-ADUser -Properties PasswordNeverExpires |
where {$_.enabled -eq $true -and $_.PasswordNeverExpires -eq $false} |
select -First 50 | %{

    Write-Host $_.UserPrincipalName
    Set-ADUser $_ -PasswordNeverExpires $true
}
```

Latest version of this snippet: <a href="https://gist.github.com/7913696">https://gist.github.com/7913696</a></pre>