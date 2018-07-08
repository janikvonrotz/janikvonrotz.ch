---
title: 'Manage the life cycle of your SCCM applicatons with PowerShell - Part 4 Remove Applications'
date: 2017-09-05T08:32:11+00:00
author: Janik von Rotz
slug: manage-the-life-cycle-of-your-sccm-applicatons-with-powershell-part-4-remove-applications
image: /wp-content/uploads/2017/08/SCCM-PowerShell-e1503910243136.jpeg
categories:
  - Configuration Manager
tags:
  - application
  - automation
  - configuration manager
  - deploy
  - deployment
  - life cycle
  - management
  - removal
  - sccm
  - script
---
"Manage the life cycle of your SCCM applications with PowerShell" is a short post series where I share my PowerShell experience with System Center Configuration Manager. In [my last post](https://janikvonrotz.ch/2017/09/01/manage-the-life-cycle-of-your-sccm-applicatons-with-powershell-part-3-deploy-applications/) I've shown you a script to distribute application content and deploy an application to its collections. In my final post I'll show you the last part of the app life cycle - the termination. 
<!--more-->

Like the last script this one is also fairly simple. Provide the name of the application you would like to remove and the script is going to remove all assets related and the application itself. For security measures the script asks for confirmation before removing anything.

**Remove-CMApplications.ps1**

```powershell
Import-Module "C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\bin\ConfigurationManager.psd1"
Import-Module ActiveDirectory
cd "$((Get-PSProvider | Where-Object {$_.Name -eq "CMSite"}).Drives.Name):"

$ApplicationName = "Visual Studio Code (1.15.1)" 
# use * for all applications

Get-CMApplication | Where-Object{ $_.LocalizedDisplayName -like $ApplicationName } | ForEach-Object {
    $Name = $_.LocalizedDisplayName
    $DeviceCollectionName = $Name + " Devices"
    $UserCollectionName = $Name + " Users"
    $ADGroupName = "DL_CM_$Name"
    
    Write-Host "`nStart removal of application $Name`n"

    $answer = Read-Host "Do you really want to remove the application $($Name)? (y/n)"

    if($answer -eq "y") {

        Write-Host "Remove application deployments."
        Get-CMApplicationDeployment | Where-Object{ ($_.ApplicationName -eq $Name) } | Remove-CMApplicationDeployment -Force

        Write-Host "Remove application package."
        Get-CMApplication -Name $Name | Remove-CMApplication -Force

        Write-Host "Remove AD group."
        Remove-ADGroup $ADGroupName -Confirm:$false

        Write-Host "Remove device and user collections"
        Remove-CMUserCollection -Name $UserCollectionName -Force
        Remove-CMDeviceCollection -Name $DeviceCollectionName -Force
    }
}
```

The only asset which is not removed is the application folder.

PowerShell nowadays is essential when administrating Microsoft products. In the long run there probably will be Microsoft products which can only configured properly using PowerShell cmdlets (see latest Exchange server). The Support for Configuration Manager is flawless, everything done in the management console can be done using PowerShell commands.