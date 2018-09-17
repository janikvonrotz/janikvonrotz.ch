---
title: 'Manage the life cycle of your SCCM applicatons with PowerShell - Part 3 Deploy Applications'
date: 2017-09-01T09:00:58+00:00
author: Janik von Rotz
slug: manage-the-life-cycle-of-your-sccm-applicatons-with-powershell-part-3-deploy-applications
images:
  - /wp-content/uploads/2017/08/SCCM-PowerShell-e1503910243136.jpeg
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

```powershell
Import-Module "C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\bin\ConfigurationManager.psd1"
cd "$((Get-PSProvider | Where-Object {$_.Name -eq "CMSite"}).Drives.Name):"

$ApplicationName = "CM Console (1.0.0)" 
# use * for all applications
$Action = "Install"
# use Install or Uninstall
$Purpose = "Required"
# use Available or Required

$DistributionPointName = "OWSPAP08.GO4KSOW.NET"
$CMApplicationDeployment = Get-CMApplicationDeployment
Get-CMApplication | Where-Object{ $_.LocalizedDisplayName -like $ApplicationName } | ForEach-Object {
    $Name = $_.LocalizedDisplayName
    $DeviceCollectionName = "$Name Devices"
    $UserCollectionName = "$Name Users"

    Write-Host "`nDistribute and deploy application $Name`n"

    try {
        Start-CMContentDistribution -ApplicationName $Name -DistributionPointName $DistributionPointName
        Write-Host "Content distributed"
    } catch {
        Write-Warning "Content has already been distributed."
    }

    $UserCollectionDeployment = $CMApplicationDeployment | Where-Object{ ($_.ApplicationName -eq $Name) -and ($_.CollectionName -eq $UserCollectionName)}
    if(-not $UserCollectionDeployment) {
        Write-Host "Deploy to user collection."
        $UserCollectionDeployment = New-CMApplicationDeployment -Name $Name -CollectionName $UserCollectionName -DeployAction $Action -DeployPurpose $Purpose
    } else {
        Write-Warning "Application has already been distributed to the user collection."
    }

    $DeviceCollectionDeployment = $CMApplicationDeployment | Where-Object{ ($_.ApplicationName -eq $Name) -and ($_.CollectionName -eq $DeviceCollectionName)}
    if(-not $DeviceCollectionDeployment) {
        Write-Host "Deploy to device collection."
        $DeviceCollectionDeployment = New-CMApplicationDeployment -Name $Name -CollectionName $DeviceCollectionName -DeployAction $Action -DeployPurpose $Purpose
    } else {
        Write-Warning "Application has already been distributed to the device collection."
    }
}
```

The next post of this series is already the last post. Stay calm!