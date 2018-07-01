---
id: 4529
title: 'Manage the life cycle of your SCCM applicatons with PowerShell &#8211; Part 4 Remove Applications'
date: 2017-09-05T08:32:11+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4529
permalink: /2017/09/05/manage-the-life-cycle-of-your-sccm-applicatons-with-powershell-part-4-remove-applications/
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

[code lang="powershell"]
Import-Module &quot;C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\bin\ConfigurationManager.psd1&quot;
Import-Module ActiveDirectory
cd &quot;$((Get-PSProvider | Where-Object {$_.Name -eq &quot;CMSite&quot;}).Drives.Name):&quot;

$ApplicationName = &quot;Visual Studio Code (1.15.1)&quot; 
# use * for all applications

Get-CMApplication | Where-Object{ $_.LocalizedDisplayName -like $ApplicationName } | ForEach-Object {
    $Name = $_.LocalizedDisplayName
    $DeviceCollectionName = $Name + &quot; Devices&quot;
    $UserCollectionName = $Name + &quot; Users&quot;
    $ADGroupName = &quot;DL_CM_$Name&quot;
    
    Write-Host &quot;`nStart removal of application $Name`n&quot;

    $answer = Read-Host &quot;Do you really want to remove the application $($Name)? (y/n)&quot;

    if($answer -eq &quot;y&quot;) {

        Write-Host &quot;Remove application deployments.&quot;
        Get-CMApplicationDeployment | Where-Object{ ($_.ApplicationName -eq $Name) } | Remove-CMApplicationDeployment -Force

        Write-Host &quot;Remove application package.&quot;
        Get-CMApplication -Name $Name | Remove-CMApplication -Force

        Write-Host &quot;Remove AD group.&quot;
        Remove-ADGroup $ADGroupName -Confirm:$false

        Write-Host &quot;Remove device and user collections&quot;
        Remove-CMUserCollection -Name $UserCollectionName -Force
        Remove-CMDeviceCollection -Name $DeviceCollectionName -Force
    }
}
[/code]

The only asset which is not removed is the application folder.

PowerShell nowadays is essential when administrating Microsoft products. In the long run there probably will be Microsoft products which can only configured properly using PowerShell cmdlets (see latest Exchange server). The Support for Configuration Manager is flawless, everything done in the management console can be done using PowerShell commands.