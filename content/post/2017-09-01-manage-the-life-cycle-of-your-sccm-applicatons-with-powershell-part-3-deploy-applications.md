---
id: 4513
title: 'Manage the life cycle of your SCCM applicatons with PowerShell &#8211; Part 3 Deploy Applications'
date: 2017-09-01T09:00:58+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4513
permalink: /2017/09/01/manage-the-life-cycle-of-your-sccm-applicatons-with-powershell-part-3-deploy-applications/
image: /wp-content/uploads/2017/08/SCCM-PowerShell-e1503910243136.jpeg
categories:
  - Configuration Manager
tags:
  - "1702"
  - application
  - automation
  - configuration manager
  - current branch
  - deployment
  - life cycle
  - management
  - powershell
  - sccm
  - script
  - system center configuration manager
---
"Manage the life cycle of your SCCM applications with PowerShell" is a short post series where I share my PowerShell experience with System Center Configuration Manager. In [my last post](https://janikvonrotz.ch/2017/08/29/manage-the-life-cycle-of-your-sccm-applicatons-with-powershell-part-2-create-applications) I've shown you a script that creates applications and all assets required to deploy it. This time I have a script to distribute the application content and deploy the application to its collections.
<!--more-->

The script is fairly simple. It cycles through the applications catalog and checks whether they already have been distributed or deployed to its collection, if not it does so.

**Deploy-CMApplications.ps1**

[code lang="powershell"]
Import-Module &quot;C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\bin\ConfigurationManager.psd1&quot;
cd &quot;$((Get-PSProvider | Where-Object {$_.Name -eq &quot;CMSite&quot;}).Drives.Name):&quot;

$ApplicationName = &quot;CM Console (1.0.0)&quot; 
# use * for all applications
$Action = &quot;Install&quot;
# use Install or Uninstall
$Purpose = &quot;Required&quot;
# use Available or Required

$DistributionPointName = &quot;OWSPAP08.GO4KSOW.NET&quot;
$CMApplicationDeployment = Get-CMApplicationDeployment
Get-CMApplication | Where-Object{ $_.LocalizedDisplayName -like $ApplicationName } | ForEach-Object {
    $Name = $_.LocalizedDisplayName
    $DeviceCollectionName = &quot;$Name Devices&quot;
    $UserCollectionName = &quot;$Name Users&quot;

    Write-Host &quot;`nDistribute and deploy application $Name`n&quot;

    try {
        Start-CMContentDistribution -ApplicationName $Name -DistributionPointName $DistributionPointName
        Write-Host &quot;Content distributed&quot;
    } catch {
        Write-Warning &quot;Content has already been distributed.&quot;
    }

    $UserCollectionDeployment = $CMApplicationDeployment | Where-Object{ ($_.ApplicationName -eq $Name) -and ($_.CollectionName -eq $UserCollectionName)}
    if(-not $UserCollectionDeployment) {
        Write-Host &quot;Deploy to user collection.&quot;
        $UserCollectionDeployment = New-CMApplicationDeployment -Name $Name -CollectionName $UserCollectionName -DeployAction $Action -DeployPurpose $Purpose
    } else {
        Write-Warning &quot;Application has already been distributed to the user collection.&quot;
    }

    $DeviceCollectionDeployment = $CMApplicationDeployment | Where-Object{ ($_.ApplicationName -eq $Name) -and ($_.CollectionName -eq $DeviceCollectionName)}
    if(-not $DeviceCollectionDeployment) {
        Write-Host &quot;Deploy to device collection.&quot;
        $DeviceCollectionDeployment = New-CMApplicationDeployment -Name $Name -CollectionName $DeviceCollectionName -DeployAction $Action -DeployPurpose $Purpose
    } else {
        Write-Warning &quot;Application has already been distributed to the device collection.&quot;
    }
}
[/code]

The next post of this series is already the last post. Stay calm!