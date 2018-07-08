---
title: 'Manage the life cycle of your SCCM applicatons with PowerShell - Part 2 Create Applications'
date: 2017-08-29T15:07:47+00:00
author: Janik von Rotz
slug: manage-the-life-cycle-of-your-sccm-applicatons-with-powershell-part-2-create-applications
image: /wp-content/uploads/2017/08/SCCM-PowerShell-e1503910243136.jpeg
categories:
  - Configuration Manager
tags:
  - "1702"
  - advanced
  - application
  - automation
  - configuration manager
  - current branch
  - deployment
  - installation
  - powershell
  - sccm
  - script
  - system center configuration manager
---
"Manage the life cycle of your SCCM applications with PowerShell" is a short post series where I share my PowerShell experience with System Center Configuration Manager. In [my last post](https://janikvonrotz.ch/2017/08/29/manage-the-life-cycle-of-your-sccm-applicatons-with-powershell-part-1-service-accounts-and-package-source/) I've showed you a script that creates the package source folder structure and another that adds the service users for SCCM. As mentioned these scripts have only been published for a better understanding of the follow-up scripts.
<!--more-->

This time I would like to present you the script that adds new applications to the SCCM catalog. Here are the main features of the script:

* Populates the folder structure for the registered applications.
* Creates a user and device collection for each application.
* Supports MSI, Executables and Scripted Installers.
* Configures deployment types for each application.
* Maps collections with an AD security group.
* Add application supersedence if configured.

The script uses a config file. In there all information are stored required to create an application. Let's have a look:

**AppConfigs.ps1**

```powershell
$AppConfigs = @{
    Name = "7-Zip"
    Description = "7-Zip ist ein freies Datenkompressionsprogramm mit einer hohen Kompressionsrate."
    Version = "17.00 beta"
    Category = "File Manager"
    Company = "7-Zip"
    Publisher = "Igor Pavlov"
    Url = "http://www.7-zip.de/"
    Deploy = @{
        Type = "msi"
        Install = "7z1700-x64.msi"
        InstallationBehaviorType = "InstallForSystem"
        LogonRequirementType = "WhereOrNotUserLoggedOn"
    }
    Update = @{
        Supersedes = "7-Zip (16.04)"
        IsUninstall = $true
    }
},
@{
    Name = "CM Console"
    Description = "System Center Configuration Manager Console."
    Version = "1.0.0"
    Category = "Configuration Manager"
    Company = "Microsoft"
    Publisher = "Microsoft"
    Url = "https://www.microsoft.com/en-us/cloud-platform/system-center-configuration-manager"
    Deploy = @{
        Type = "script"
        Install = 'ConsoleSetup.exe /q TargetDir="C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole" EnableSQM=1 DefaultSiteServerName=owspap08.go4ksow.net'
        Uninstall = 'ConsoleSetup.exe /q TargetDir="C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole" /uninstall'
        InstallationBehaviorType = "InstallForSystem"
        LogonRequirementType = "WhereOrNotUserLoggedOn"
        DetectionScript =  @'
if (Test-Path "C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\bin\Microsoft.ConfigurationManagement.exe") {
        Write-Host "Installed"
} else {
}
'@
    }
},
@{
    Name = "7-Zip"
    Description = "7-Zip ist ein freies Datenkompressionsprogramm mit einer hohen Kompressionsrate."
    Version = "16.04"
    Category = "File Manager"
    Company = "7-Zip"
    Publisher = "Igor Pavlov"
    Url = "http://www.7-zip.de/"
    Deploy = @{
        Type = "msi"
        Install = "7z1604-x64.msi"
        InstallationBehaviorType = "InstallForSystem"
        LogonRequirementType = "WhereOrNotUserLoggedOn"
    }
},
@{
    Name = "VLC Script"
    Description = "VLC is a free and open source cross-platform multimedia player and framework that plays most multimedia files as well as DVDs, Audio CDs, VCDs, and various streaming protocols."
    Version = "2.2.6"
    Category = "Video Player"
    Company = "VideoLAN"
    Publisher = "VideoLAN Organization"
    Url = "https://www.videolan.org/"
    Deploy = @{
        Type = "powershell"
        Install = "powershell.exe -ExecutionPolicy Bypass -File .\Application.ps1 -Install"
        Uninstall = "powershell.exe -ExecutionPolicy Bypass -File .\Application.ps1 -Uninstall"
        InstallationBehaviorType = "InstallForSystem"
        LogonRequirementType = "WhereOrNotUserLoggedOn"
        DetectionScript =  @'
if (Test-Path "C:\Program Files (x86)\VideoLAN\VLC\vlc.exe") {
        Write-Host "Installed"
} else {
}
'@
    }
},
@{
    Name = "VLC"
    Description = "VLC is a free and open source cross-platform multimedia player and framework that plays most multimedia files as well as DVDs, Audio CDs, VCDs, and various streaming protocols."
    Version = "2.2.6"
    Category = "Video Player"
    Company = "VideoLAN"
    Publisher = "VideoLAN Organization"
    Url = "https://www.videolan.org/"
    Deploy = @{
        Type = "script"
        Install = "vlc-2.2.6-win32.exe /S"
        Uninstall = '"C:\Program Files (x86)\VideoLAN\VLC\uninstall.exe" /S'
        InstallationBehaviorType = "InstallForSystem"
        LogonRequirementType = "WhereOrNotUserLoggedOn"
        DetectionScript =  @'
if (Test-Path "C:\Program Files (x86)\VideoLAN\VLC\vlc.exe") {
        Write-Host "Installed"
} else {
}
'@
    }
},
@{
    Name = "Greenshot"
    Description = "Greenshot is the most awesome tool for making screenshots you can get on your Windows PC."
    Version = "1.2.9.129"
    Category = "Utility"
    Company = "Greenshot"
    Publisher = "Jens Klingen"
    Url = "http://getgreenshot.org/"
    Deploy = @{
        Type = "script"
        Install = 'Greenshot-INSTALLER-1.2.9.129-RELEASE.exe /VERYSILENT /NORESTART'
        Uninstall = '"C:\Program Files\Greenshot\unins000.exe" /VERYSILENT /NORESTART'
        InstallationBehaviorType = "InstallForSystem"
        LogonRequirementType = "WhereOrNotUserLoggedOn"
        DetectionScript =  @'
if (Test-Path "C:\Program Files\Greenshot\Greenshot.exe") {
        Write-Host "Installed"
} else {
}
'@
    }
}
```

As you cans see the config supports different kind of deployment types (script, msi, powershell) and supersedence. When it comes to deployment types, the script can be extended easily.

The config file is used by the script to create and configure the applications.

**Add-SCCMApplications.ps1**

```powershell
Import-Module "C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\bin\ConfigurationManager.psd1"
Import-Module ActiveDirectory
cd "$((Get-PSProvider | Where-Object {$_.Name -eq "CMSite"}).Drives.Name):"

$CM_OU = "OU=SCCM,OU=AccessGroups,OU=Admin,DC=go4ksow,DC=net"
$PackageSource = "D:\SCCM\Applications\"
$HostName = "owspap08"
. "$PSScriptRoot\AppConfigs.ps1"

$Domain = Get-ADDomain
$CMApplications = Get-CMApplication
$CMCategories = Get-CMCategory -CategoryType AppCategories
$CMDeviceCollections = Get-CMDeviceCollection
$CMUserCollections = Get-CMUserCollection

$AppConfigs | ForEach-Object {
    $AppConfig = $_
    $AppConfig.AppName = $AppConfig.Name + " (" + $AppConfig.Version + ")"
    $ContentLocation = "\\$HostName\PackageSource\Applications\$($AppConfig.Company)\$($AppConfig.Name)\$($AppConfig.Version)"
    $ADGroupName = "DL_CM_$($AppConfig.AppName)"
    $DeviceCollectionName =  $AppConfig.AppName + " Devices"
    $UserCollectionName =  $AppConfig.AppName + " Users"

    Write-Host "`nAdd CM Application $($AppConfig.AppName)`n"

    $App = $CMApplications | Where-Object{ $_.LocalizedDisplayName -eq $AppConfig.AppName}
    if(-not $App) {
        Write-Host "Create the application: $($AppConfig.AppName)."
        $IconLocationFile = Join-Path $PackageSource "icon.jpg"
        $App = New-CMApplication -Name $AppConfig.AppName `
            -LocalizedDescription $AppConfig.Description `
            -SoftwareVersion $AppConfig.Version `
            -Publisher $AppConfig.Publisher `
            -LinkText $AppConfig.Url `
            -IconLocationFile $IconLocationFile
    } else {
        Write-Warning "Application: $($AppConfig.AppName) already exists."
    }

    $Category = $CMCategories | Where-Object{ $_.LocalizedCategoryInstanceName -eq $AppConfig.Category }
    if(-not $Category) {
        Write-Host "Create the application category: $($AppConfig.Category)."
        $Category = New-CMCategory -CategoryType AppCategories -Name $AppConfig.Category
    } else {
        Write-Warning "Category: $($AppConfig.Category) already exists."
    }

    $CategoryIsSet = $null
    $CategoryIsSet = $App.LocalizedCategoryInstanceNames[0] -eq $Category.LocalizedCategoryInstanceName
    if(-not $CategoryIsSet) {
        Write-Host "Set application category."
        $App | Set-CMApplication -AppCategory $Category.LocalizedCategoryInstanceName
    } else {
        Write-Warning "Application category is already set."
    }

    $Path = $null
    $Path = (Join-Path $PackageSource $AppConfig.Company)
    if(-not (Test-Path -Path $Path)) {
        Write-Host "Create company application folder."
        New-Item -Path $Path -ItemType Directory | Out-Null
    }
    $Path = (Join-Path $Path $AppConfig.Name)
    if(-not (Test-Path -Path $Path)) {
        Write-Host "Create application folder."
        New-Item -Path $Path  -ItemType Directory | Out-Null
    }
    $Path = (Join-Path $Path $AppConfig.Version)
    if(-not (Test-Path -Path $Path)) {
        Write-Host "Create application version folder."
        New-Item -Path $Path  -ItemType Directory | Out-Null
    }

    $ADGroup = $null
    $ADGroup = Get-ADGroup $ADGroupName
    if(-not $ADGroup){
        Write-Host "Add AD group: $ADGroupName."
        $ADGroup = New-ADGroup -Name $ADGroupName -GroupScope DomainLocal -Path $CM_OU
    } else {
        Write-Warning "AD group: $ADGroupName already exists."
    }

    if($AppConfig.Deploy.type -eq "script") {

        $deployType = Get-CMDeploymentType -ApplicationName $AppConfig.AppName
        if(-not $DeployType){
            Write-Host "Add script deployment type."

            $param = @{
                ApplicationName = $AppConfig.AppName
                DeploymentTypeName = $AppConfig.AppName
                ScriptText = $AppConfig.Deploy.DetectionScript
                ScriptLanguage = "PowerShell"
                ContentLocation = $ContentLocation
                InstallCommand = $AppConfig.Deploy.Install
                UninstallCommand = $AppConfig.Deploy.Uninstall
                InstallationBehaviorType = $AppConfig.Deploy.InstallationBehaviorType
                LogonRequirementType = $AppConfig.Deploy.LogonRequirementType
                ForceScriptDetection32Bit = $true
            }
           $DeployType = Add-CMScriptDeploymentType @param
        } else {
            Write-Warning "Script deployment type already exists."
        }
    }

    if($AppConfig.Deploy.type -eq "msi") {

        $DeployType = Get-CMDeploymentType -ApplicationName $AppConfig.AppName
        if(-not $DeployType){
            Write-Host "Add default deployment type."

            $param = @{
                ApplicationName = $AppConfig.AppName
                DeploymentTypeName = $AppConfig.AppName
                ContentLocation = Join-path $ContentLocation $AppConfig.Deploy.Install
                InstallationBehaviorType = $AppConfig.Deploy.InstallationBehaviorType
                LogonRequirementType = $AppConfig.Deploy.LogonRequirementType
                Force = $true  
            }
            $DeployType = Add-CMMsiDeploymentType @param
        } else {
            Write-Warning "Script deployment type already exists."
        }
    }

    if($AppConfig.Deploy.type -eq "powershell") {

        $deployType = Get-CMDeploymentType -ApplicationName $AppConfig.AppName
        if(-not $DeployType){
            Write-Host "Add powershell deployment type."

            $param = @{
                ApplicationName = $AppConfig.AppName
                DeploymentTypeName = $AppConfig.AppName
                ScriptText = $AppConfig.Deploy.DetectionScript
                ScriptLanguage = "PowerShell"
                ContentLocation = $ContentLocation
                InstallCommand = $AppConfig.Deploy.Install
                UninstallCommand = $AppConfig.Deploy.Uninstall
                InstallationBehaviorType = $AppConfig.Deploy.InstallationBehaviorType
                LogonRequirementType = $AppConfig.Deploy.LogonRequirementType
            }
           $DeployType = Add-CMScriptDeploymentType @param
        } else {
            Write-Warning "Powershell deployment type already exists."
        }

        $ScriptPath = Join-Path $Path "Application.ps1"
        if(-not (Test-Path $ScriptPath)) {
            Write-Host "Add powershell deployment script."
            @'
Param(
  [switch]$Install,
  [switch]$Uninstall
)

if($Install) {
    Start-Process -FilePath ".\vlc-2.2.6-win32.exe" -ArgumentList "/S" -Wait
}

if($Uninstall) {
    Start-Process -FilePath "C:\Program Files (x86)\VideoLAN\VLC\uninstall.exe" -ArgumentList "/S" -Wait
}
'@ | Set-Content -Path $ScriptPath
        } else {
            Write-Warning "PowerShell deployment script already exists."
        }
    }

    $LimitingCollection = $CMDeviceCollections | Where-Object{ $_.Name -eq "All Systems" }
    $DeviceCollection = $CMDeviceCollections | Where-Object{ $_.Name -eq $DeviceCollectionName }
    if(-not $DeviceCollection) {
        Write-Host "Create the device collection."
        $DeviceCollection = New-CMDeviceCollection -Name $DeviceCollectionName -LimitingCollectionId $LimitingCollection.CollectionID -RefreshType 4 -Comment "Application Collection"
    } else {
        Write-Warning "Device collection already exists."
    }

    $QueryRule = 'select *  from  SMS_R_System where SMS_R_System.SecurityGroupName = "' + $Domain.Name + '\\' + $ADGroup.Name + '"'
    $Rule = Get-CMDeviceCollectionQueryMembershipRule -Name $DeviceCollectionName
    if(-not $Rule -and $ADGroup) {
        Write-Host "Add device collection membership rule."
        Add-CMDeviceCollectionQueryMembershipRule -CollectionName $DeviceCollectionName -RuleName "Active Directory Membership" -QueryExpression $QueryRule
    } elseif(-not $ADGroup) {
        Write-Warning "Could not create device colleciton membership rule as AD groups does not exist."
    } else {
        Write-Warning "Device collection membership rule already exists."
    }

    $LimitingCollection = $null
    $LimitingCollection = $CMUserCollections | Where-Object{ $_.Name -eq "All Users" }
    $UserCollection = $CMUserCollections | Where-Object{ $_.Name -eq $UserCollectionName }
    if(-not $UserCollection) {
        Write-Host "Create the user collection."
        $UserCollection = New-CMUserCollection -Name $UserCollectionName -LimitingCollectionId $LimitingCollection.CollectionID -RefreshType 4 -Comment "Application Collection"
    } else {
        Write-Warning "User collection already exists."
    }

    $QueryRule = $null
    $QueryRule = 'select *  from  SMS_R_User where SMS_R_User.SecurityGroupName = "' + $Domain.Name + '\\' + $ADGroup.Name + '"'
    $Rule = $null
    $Rule = Get-CMUserCollectionQueryMembershipRule -Name $UserCollectionName
    if(-not $Rule -and $ADGroup) {
        Write-Host "Add user collection membership rule."
        Add-CMUserCollectionQueryMembershipRule -CollectionName $UserCollectionName -RuleName "Active Directory Membership" -QueryExpression $QueryRule
    } elseif(-not $ADGroup) {
        Write-Warning "Could not create user colleciton membership rule as AD groups does not exist."
    } else {
        Write-Warning "User collection membership rule already exists."
    }

    if($AppConfig.Update) {
        if(-not $App.IsSuperseding) {
            Write-Host "Add application supersedence for $($AppConfig.Update.Supersedes)."
            $SupersedingDeploymentType = Get-CMDeploymentType -DeploymentTypeName $AppConfig.AppName -ApplicationName $AppConfig.AppName
            $SupersededDeploymentType = Get-CMDeploymentType -DeploymentTypeName $AppConfig.Update.Supersedes -ApplicationName $AppConfig.Update.Supersedes
            $DeploymentTypeSupersedence = Add-CMDeploymentTypeSupersedence -SupersedingDeploymentType $SupersedingDeploymentType -SupersededDeploymentType $SupersededDeploymentType -IsUninstall:$AppConfig.Update.IsUninstall
        } else {
             Write-Warning "This application already supersedes another application."
        }
    }
}
```

The script creates all assets required to deploy an application and reduces the probability of misconfiguration drastically. Here's a screenshot of the assets for the VLC player package:

[![Untitled](/wp-content/uploads/2017/08/SCCM-application-assets.png)](/wp-content/uploads/2017/08/SCCM-application-assets.png)

The application package has PowerShell deployment type. Instead of an exectable a PowerShell is called to install the Application. Using a script allows to make additional configurations before and after the installation or uinstallation of the package.

In my next post I will share the script for deploying the created applications. Stay focused!