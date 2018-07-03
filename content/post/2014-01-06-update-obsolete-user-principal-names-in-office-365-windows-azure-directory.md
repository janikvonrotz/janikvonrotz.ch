---
id: 891
title: Update Obsolete User Principal Names in Office 365 Windows Azure Directory
date: 2014-01-06T13:02:55+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=891
permalink: /2014/01/06/update-obsolete-user-principal-names-in-office-365-windows-azure-directory/
dsq_thread_id:
  - "2095263651"
image: /wp-content/uploads/2014/01/Windows-Azure-Directory-Services-672x200.png
categories:
  - Office 365
  - PowerShell
tags:
  - active
  - attribute
  - azure
  - directory
  - flow
  - name
  - powershell
  - principal
  - script
  - services
  - synchronisation
  - update
  - user
---
It could happen that the directory sync service (DirSync) doesn't sync the users UserPrincipalName correctly.

I had an issue where the UserPrincipalName from a user in the Office 365 windows azure directory has been made based on the user's sAMAccountname. This wouldn't be problem if as long the sAMAccountname is the as same as the UserPrincipalName, but as you can guess this is not everywhere the case.

<!--more-->

First I checked the attribute flow of the synchronization job and as you can in see in picture below the DirSync service will update the attribute first by the users DN, then by it's sAMAccountname and at least by the UserPrincipalName .
![DirSync UPN Config](https://janikvonrotz.ch/wp-content/uploads/2014/01/DirSync-UPN-Config-1024x558.jpg)

I'm not quite shure wether the sync problem is caused by this configuration or not, but I don't recommend to edit these rules, in order the get the UserPrincipalName synced correctly.

To update the UPN I've written the following script, I'll compare the user attributes from the ActiveDirectory and the AzureDirectory. If a UPN doesn't match it'll be overwritten with the ActiveDirectory UPN.

```ps

#--------------------------------------------------#
# settings
#--------------------------------------------------#

$OU = "OU=vblusers2,DC=vbl,DC=ch"

#--------------------------------------------------#
# modules
#--------------------------------------------------#

Import-Module MSOnline
Import-Module MSOnlineExtended
Import-Module ActiveDirectory

#--------------------------------------------------#
# main
#--------------------------------------------------#

$ADUsers = Get-ADUser -Filter * -SearchBase $OU -Properties GivenName, Surname, DisplayName

$Credential = Import-PSCredential $(Get-ChildItem -Path $PSconfigs.Path -Filter "Office365.credentials.config.xml" -Recurse).FullName
Connect-MsolService -Credential $Credential
$MsolUsers = Get-MsolUser -All

$MsolUsers | %{

    $MsolUser = $_
    $ADUsers | where{($_.GivenName -eq $MsolUser.FirstName) -and
        ($_.Surname -eq $MsolUser.LastName) -and
        ($_.DisplayName -eq $MsolUser.DisplayName) -and
        ($_.UserPrincipalName -ne $MsolUser.UserPrincipalName)
    } | %{

        Write-Host "Change UserPrincipalName for: $($MsolUser.UserPrincipalName) to: $($_.UserPrincipalName)"
        Set-MsolUserPrincipalName -UserPrincipalName $MsolUser.UserPrincipalName -NewUserPrincipalName $_.UserPrincipalName

    }
}

```

<h1>Requirements</h1>

<ul>
    <li>PowerShell Active Directory and Microsoft Online modules</li>
    <li>Optional: functions from my project: <a href="https://github.com/janikvonrotz/PowerShell-Profile">https://github.com/janikvonrotz/PowerShell-Profile</a></li>
</ul>

<h1>Source</h1>

Latest version of this script: <a href="https://gist.github.com/8281095">https://gist.github.com/8281095</a>