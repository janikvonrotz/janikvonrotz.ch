---
title: Active Directory User Reporting
date: 2013-07-25T10:49:32+00:00
author: Janik von Rotz
permalink: /2013/07/25/active-directory-user-reporting/
dsq_thread_id:
  - "1529885769"
image: /wp-content/uploads/2013/07/PowerShell.png
categories:
  - PowerShell
tags:
  - activedirectory
  - powershell
  - reporting
  - scripting
  - user
  - windows
---
This is a simple example of how to create a report of your Active Directory users.
The first command imports the PowerShell Active Directory module, which should be installed by default, otherwhise do this:

![Install PowerShell Active Directory Module](/wp-content/uploads/2013/07/2013-07-25-11_43_24-Windows-Funktionen.png)

```powershell
Import-Module ActiveDirectory
Get-ADUser -Filter {EmailAddress -like "*"} -Properties * | select DisplayName, GivenName, Name, Surname, mail, SamAccountName, Department, Title, extensionAttribute1, extensionAttribute2 | Out-GridView
```

And the second command creates a simple report.