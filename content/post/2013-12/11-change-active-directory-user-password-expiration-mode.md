---
title: Change Active Directory User Password Expiration Mode
date: 2013-12-11T17:35:45+00:00
author: Janik Vonrotz
slug: change-active-directory-user-password-expiration-mode
images:
  - /wp-content/uploads/2013/12/PowerShell-and-ActiveDirectory.png
categories:
  - Microsoft infrastructure
tags:
  - activedirectory
  - expiration
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