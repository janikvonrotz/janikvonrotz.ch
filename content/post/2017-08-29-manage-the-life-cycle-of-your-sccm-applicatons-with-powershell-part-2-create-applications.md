---
id: 4496
title: 'Manage the life cycle of your SCCM applicatons with PowerShell &#8211; Part 2 Create Applications'
date: 2017-08-29T15:07:47+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4496
permalink: /2017/08/29/manage-the-life-cycle-of-your-sccm-applicatons-with-powershell-part-2-create-applications/
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

[code lang="powershell"]
$AppConfigs = @{
    Name = &quot;7-Zip&quot;
    Description = &quot;7-Zip ist ein freies Datenkompressionsprogramm mit einer hohen Kompressionsrate.&quot;
    Version = &quot;17.00 beta&quot;
    Category = &quot;File Manager&quot;
    Company = &quot;7-Zip&quot;
    Publisher = &quot;Igor Pavlov&quot;
    Url = &quot;http://www.7-zip.de/&quot;
    Deploy = @{
        Type = &quot;msi&quot;
        Install = &quot;7z1700-x64.msi&quot;
        InstallationBehaviorType = &quot;InstallForSystem&quot;
        LogonRequirementType = &quot;WhereOrNotUserLoggedOn&quot;
    }
    Update = @{
        Supersedes = &quot;7-Zip (16.04)&quot;
        IsUninstall = $true
    }
},
@{
    Name = &quot;CM Console&quot;
    Description = &quot;System Center Configuration Manager Console.&quot;
    Version = &quot;1.0.0&quot;
    Category = &quot;Configuration Manager&quot;
    Company = &quot;Microsoft&quot;
    Publisher = &quot;Microsoft&quot;
    Url = &quot;https://www.microsoft.com/en-us/cloud-platform/system-center-configuration-manager&quot;
    Deploy = @{
        Type = &quot;script&quot;
        Install = 'ConsoleSetup.exe /q TargetDir=&quot;C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole&quot; EnableSQM=1 DefaultSiteServerName=owspap08.go4ksow.net'
        Uninstall = 'ConsoleSetup.exe /q TargetDir=&quot;C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole&quot; /uninstall'
        InstallationBehaviorType = &quot;InstallForSystem&quot;
        LogonRequirementType = &quot;WhereOrNotUserLoggedOn&quot;
        DetectionScript =  @'
if (Test-Path &quot;C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\bin\Microsoft.ConfigurationManagement.exe&quot;) {
        Write-Host &quot;Installed&quot;
} else {
}
'@
    }
},
@{
    Name = &quot;7-Zip&quot;
    Description = &quot;7-Zip ist ein freies Datenkompressionsprogramm mit einer hohen Kompressionsrate.&quot;
    Version = &quot;16.04&quot;
    Category = &quot;File Manager&quot;
    Company = &quot;7-Zip&quot;
    Publisher = &quot;Igor Pavlov&quot;
    Url = &quot;http://www.7-zip.de/&quot;
    Deploy = @{
        Type = &quot;msi&quot;
        Install = &quot;7z1604-x64.msi&quot;
        InstallationBehaviorType = &quot;InstallForSystem&quot;
        LogonRequirementType = &quot;WhereOrNotUserLoggedOn&quot;
    }
},
@{
    Name = &quot;VLC Script&quot;
    Description = &quot;VLC is a free and open source cross-platform multimedia player and framework that plays most multimedia files as well as DVDs, Audio CDs, VCDs, and various streaming protocols.&quot;
    Version = &quot;2.2.6&quot;
    Category = &quot;Video Player&quot;
    Company = &quot;VideoLAN&quot;
    Publisher = &quot;VideoLAN Organization&quot;
    Url = &quot;https://www.videolan.org/&quot;
    Deploy = @{
        Type = &quot;powershell&quot;
        Install = &quot;powershell.exe -ExecutionPolicy Bypass -File .\Application.ps1 -Install&quot;
        Uninstall = &quot;powershell.exe -ExecutionPolicy Bypass -File .\Application.ps1 -Uninstall&quot;
        InstallationBehaviorType = &quot;InstallForSystem&quot;
        LogonRequirementType = &quot;WhereOrNotUserLoggedOn&quot;
        DetectionScript =  @'
if (Test-Path &quot;C:\Program Files (x86)\VideoLAN\VLC\vlc.exe&quot;) {
        Write-Host &quot;Installed&quot;
} else {
}
'@
    }
},
@{
    Name = &quot;VLC&quot;
    Description = &quot;VLC is a free and open source cross-platform multimedia player and framework that plays most multimedia files as well as DVDs, Audio CDs, VCDs, and various streaming protocols.&quot;
    Version = &quot;2.2.6&quot;
    Category = &quot;Video Player&quot;
    Company = &quot;VideoLAN&quot;
    Publisher = &quot;VideoLAN Organization&quot;
    Url = &quot;https://www.videolan.org/&quot;
    Deploy = @{
        Type = &quot;script&quot;
        Install = &quot;vlc-2.2.6-win32.exe /S&quot;
        Uninstall = '&quot;C:\Program Files (x86)\VideoLAN\VLC\uninstall.exe&quot; /S'
        InstallationBehaviorType = &quot;InstallForSystem&quot;
        LogonRequirementType = &quot;WhereOrNotUserLoggedOn&quot;
        DetectionScript =  @'
if (Test-Path &quot;C:\Program Files (x86)\VideoLAN\VLC\vlc.exe&quot;) {
        Write-Host &quot;Installed&quot;
} else {
}
'@
    }
},
@{
    Name = &quot;Greenshot&quot;
    Description = &quot;Greenshot is the most awesome tool for making screenshots you can get on your Windows PC.&quot;
    Version = &quot;1.2.9.129&quot;
    Category = &quot;Utility&quot;
    Company = &quot;Greenshot&quot;
    Publisher = &quot;Jens Klingen&quot;
    Url = &quot;http://getgreenshot.org/&quot;
    Deploy = @{
        Type = &quot;script&quot;
        Install = 'Greenshot-INSTALLER-1.2.9.129-RELEASE.exe /VERYSILENT /NORESTART'
        Uninstall = '&quot;C:\Program Files\Greenshot\unins000.exe&quot; /VERYSILENT /NORESTART'
        InstallationBehaviorType = &quot;InstallForSystem&quot;
        LogonRequirementType = &quot;WhereOrNotUserLoggedOn&quot;
        DetectionScript =  @'
if (Test-Path &quot;C:\Program Files\Greenshot\Greenshot.exe&quot;) {
        Write-Host &quot;Installed&quot;
} else {
}
'@
    }
}
[/code]

As you cans see the config supports different kind of deployment types (script, msi, powershell) and supersedence. When it comes to deployment types, the script can be extended easily.

The config file is used by the script to create and configure the applications.

**Add-SCCMApplications.ps1**

[code lang="powershell"]
Import-Module &quot;C:\Program Files (x86)\Microsoft Configuration Manager\AdminConsole\bin\ConfigurationManager.psd1&quot;
Import-Module ActiveDirectory
cd &quot;$((Get-PSProvider | Where-Object {$_.Name -eq &quot;CMSite&quot;}).Drives.Name):&quot;

$CM_OU = &quot;OU=SCCM,OU=AccessGroups,OU=Admin,DC=go4ksow,DC=net&quot;
$PackageSource = &quot;D:\SCCM\Applications\&quot;
$HostName = &quot;owspap08&quot;
. &quot;$PSScriptRoot\AppConfigs.ps1&quot;

$Domain = Get-ADDomain
$CMApplications = Get-CMApplication
$CMCategories = Get-CMCategory -CategoryType AppCategories
$CMDeviceCollections = Get-CMDeviceCollection
$CMUserCollections = Get-CMUserCollection

$AppConfigs | ForEach-Object {
    $AppConfig = $_
    $AppConfig.AppName = $AppConfig.Name + &quot; (&quot; + $AppConfig.Version + &quot;)&quot;
    $ContentLocation = &quot;\\$HostName\PackageSource\Applications\$($AppConfig.Company)\$($AppConfig.Name)\$($AppConfig.Version)&quot;
    $ADGroupName = &quot;DL_CM_$($AppConfig.AppName)&quot;
    $DeviceCollectionName =  $AppConfig.AppName + &quot; Devices&quot;
    $UserCollectionName =  $AppConfig.AppName + &quot; Users&quot;

    Write-Host &quot;`nAdd CM Application $($AppConfig.AppName)`n&quot;

    $App = $CMApplications | Where-Object{ $_.LocalizedDisplayName -eq $AppConfig.AppName}
    if(-not $App) {
        Write-Host &quot;Create the application: $($AppConfig.AppName).&quot;
        $IconLocationFile = Join-Path $PackageSource &quot;icon.jpg&quot;
        $App = New-CMApplication -Name $AppConfig.AppName `
            -LocalizedDescription $AppConfig.Description `
            -SoftwareVersion $AppConfig.Version `
            -Publisher $AppConfig.Publisher `
            -LinkText $AppConfig.Url `
            -IconLocationFile $IconLocationFile
    } else {
        Write-Warning &quot;Application: $($AppConfig.AppName) already exists.&quot;
    }

    $Category = $CMCategories | Where-Object{ $_.LocalizedCategoryInstanceName -eq $AppConfig.Category }
    if(-not $Category) {
        Write-Host &quot;Create the application category: $($AppConfig.Category).&quot;
        $Category = New-CMCategory -CategoryType AppCategories -Name $AppConfig.Category
    } else {
        Write-Warning &quot;Category: $($AppConfig.Category) already exists.&quot;
    }

    $CategoryIsSet = $null
    $CategoryIsSet = $App.LocalizedCategoryInstanceNames[0] -eq $Category.LocalizedCategoryInstanceName
    if(-not $CategoryIsSet) {
        Write-Host &quot;Set application category.&quot;
        $App | Set-CMApplication -AppCategory $Category.LocalizedCategoryInstanceName
    } else {
        Write-Warning &quot;Application category is already set.&quot;
    }

    $Path = $null
    $Path = (Join-Path $PackageSource $AppConfig.Company)
    if(-not (Test-Path -Path $Path)) {
        Write-Host &quot;Create company application folder.&quot;
        New-Item -Path $Path -ItemType Directory | Out-Null
    }
    $Path = (Join-Path $Path $AppConfig.Name)
    if(-not (Test-Path -Path $Path)) {
        Write-Host &quot;Create application folder.&quot;
        New-Item -Path $Path  -ItemType Directory | Out-Null
    }
    $Path = (Join-Path $Path $AppConfig.Version)
    if(-not (Test-Path -Path $Path)) {
        Write-Host &quot;Create application version folder.&quot;
        New-Item -Path $Path  -ItemType Directory | Out-Null
    }

    $ADGroup = $null
    $ADGroup = Get-ADGroup $ADGroupName
    if(-not $ADGroup){
        Write-Host &quot;Add AD group: $ADGroupName.&quot;
        $ADGroup = New-ADGroup -Name $ADGroupName -GroupScope DomainLocal -Path $CM_OU
    } else {
        Write-Warning &quot;AD group: $ADGroupName already exists.&quot;
    }

    if($AppConfig.Deploy.type -eq &quot;script&quot;) {

        $deployType = Get-CMDeploymentType -ApplicationName $AppConfig.AppName
        if(-not $DeployType){
            Write-Host &quot;Add script deployment type.&quot;

            $param = @{
                ApplicationName = $AppConfig.AppName
                DeploymentTypeName = $AppConfig.AppName
                ScriptText = $AppConfig.Deploy.DetectionScript
                ScriptLanguage = &quot;PowerShell&quot;
                ContentLocation = $ContentLocation
                InstallCommand = $AppConfig.Deploy.Install
                UninstallCommand = $AppConfig.Deploy.Uninstall
                InstallationBehaviorType = $AppConfig.Deploy.InstallationBehaviorType
                LogonRequirementType = $AppConfig.Deploy.LogonRequirementType
                ForceScriptDetection32Bit = $true
            }
           $DeployType = Add-CMScriptDeploymentType @param
        } else {
            Write-Warning &quot;Script deployment type already exists.&quot;
        }
    }

    if($AppConfig.Deploy.type -eq &quot;msi&quot;) {

        $DeployType = Get-CMDeploymentType -ApplicationName $AppConfig.AppName
        if(-not $DeployType){
            Write-Host &quot;Add default deployment type.&quot;

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
            Write-Warning &quot;Script deployment type already exists.&quot;
        }
    }

    if($AppConfig.Deploy.type -eq &quot;powershell&quot;) {

        $deployType = Get-CMDeploymentType -ApplicationName $AppConfig.AppName
        if(-not $DeployType){
            Write-Host &quot;Add powershell deployment type.&quot;

            $param = @{
                ApplicationName = $AppConfig.AppName
                DeploymentTypeName = $AppConfig.AppName
                ScriptText = $AppConfig.Deploy.DetectionScript
                ScriptLanguage = &quot;PowerShell&quot;
                ContentLocation = $ContentLocation
                InstallCommand = $AppConfig.Deploy.Install
                UninstallCommand = $AppConfig.Deploy.Uninstall
                InstallationBehaviorType = $AppConfig.Deploy.InstallationBehaviorType
                LogonRequirementType = $AppConfig.Deploy.LogonRequirementType
            }
           $DeployType = Add-CMScriptDeploymentType @param
        } else {
            Write-Warning &quot;Powershell deployment type already exists.&quot;
        }

        $ScriptPath = Join-Path $Path &quot;Application.ps1&quot;
        if(-not (Test-Path $ScriptPath)) {
            Write-Host &quot;Add powershell deployment script.&quot;
            @'
Param(
  [switch]$Install,
  [switch]$Uninstall
)

if($Install) {
    Start-Process -FilePath &quot;.\vlc-2.2.6-win32.exe&quot; -ArgumentList &quot;/S&quot; -Wait
}

if($Uninstall) {
    Start-Process -FilePath &quot;C:\Program Files (x86)\VideoLAN\VLC\uninstall.exe&quot; -ArgumentList &quot;/S&quot; -Wait
}
'@ | Set-Content -Path $ScriptPath
        } else {
            Write-Warning &quot;PowerShell deployment script already exists.&quot;
        }
    }

    $LimitingCollection = $CMDeviceCollections | Where-Object{ $_.Name -eq &quot;All Systems&quot; }
    $DeviceCollection = $CMDeviceCollections | Where-Object{ $_.Name -eq $DeviceCollectionName }
    if(-not $DeviceCollection) {
        Write-Host &quot;Create the device collection.&quot;
        $DeviceCollection = New-CMDeviceCollection -Name $DeviceCollectionName -LimitingCollectionId $LimitingCollection.CollectionID -RefreshType 4 -Comment &quot;Application Collection&quot;
    } else {
        Write-Warning &quot;Device collection already exists.&quot;
    }

    $QueryRule = 'select *  from  SMS_R_System where SMS_R_System.SecurityGroupName = &quot;' + $Domain.Name + '\\' + $ADGroup.Name + '&quot;'
    $Rule = Get-CMDeviceCollectionQueryMembershipRule -Name $DeviceCollectionName
    if(-not $Rule -and $ADGroup) {
        Write-Host &quot;Add device collection membership rule.&quot;
        Add-CMDeviceCollectionQueryMembershipRule -CollectionName $DeviceCollectionName -RuleName &quot;Active Directory Membership&quot; -QueryExpression $QueryRule
    } elseif(-not $ADGroup) {
        Write-Warning &quot;Could not create device colleciton membership rule as AD groups does not exist.&quot;
    } else {
        Write-Warning &quot;Device collection membership rule already exists.&quot;
    }

    $LimitingCollection = $null
    $LimitingCollection = $CMUserCollections | Where-Object{ $_.Name -eq &quot;All Users&quot; }
    $UserCollection = $CMUserCollections | Where-Object{ $_.Name -eq $UserCollectionName }
    if(-not $UserCollection) {
        Write-Host &quot;Create the user collection.&quot;
        $UserCollection = New-CMUserCollection -Name $UserCollectionName -LimitingCollectionId $LimitingCollection.CollectionID -RefreshType 4 -Comment &quot;Application Collection&quot;
    } else {
        Write-Warning &quot;User collection already exists.&quot;
    }

    $QueryRule = $null
    $QueryRule = 'select *  from  SMS_R_User where SMS_R_User.SecurityGroupName = &quot;' + $Domain.Name + '\\' + $ADGroup.Name + '&quot;'
    $Rule = $null
    $Rule = Get-CMUserCollectionQueryMembershipRule -Name $UserCollectionName
    if(-not $Rule -and $ADGroup) {
        Write-Host &quot;Add user collection membership rule.&quot;
        Add-CMUserCollectionQueryMembershipRule -CollectionName $UserCollectionName -RuleName &quot;Active Directory Membership&quot; -QueryExpression $QueryRule
    } elseif(-not $ADGroup) {
        Write-Warning &quot;Could not create user colleciton membership rule as AD groups does not exist.&quot;
    } else {
        Write-Warning &quot;User collection membership rule already exists.&quot;
    }

    if($AppConfig.Update) {
        if(-not $App.IsSuperseding) {
            Write-Host &quot;Add application supersedence for $($AppConfig.Update.Supersedes).&quot;
            $SupersedingDeploymentType = Get-CMDeploymentType -DeploymentTypeName $AppConfig.AppName -ApplicationName $AppConfig.AppName
            $SupersededDeploymentType = Get-CMDeploymentType -DeploymentTypeName $AppConfig.Update.Supersedes -ApplicationName $AppConfig.Update.Supersedes
            $DeploymentTypeSupersedence = Add-CMDeploymentTypeSupersedence -SupersedingDeploymentType $SupersedingDeploymentType -SupersededDeploymentType $SupersededDeploymentType -IsUninstall:$AppConfig.Update.IsUninstall
        } else {
             Write-Warning &quot;This application already supersedes another application.&quot;
        }
    }
}
[/code]

The script creates all assets required to deploy an application and reduces the probability of misconfiguration drastically. Here's a screenshot of the assets for the VLC player package:

<a href="https://janikvonrotz.ch/wp-content/uploads/2017/08/SCCM-application-assets.png"><img src="https://janikvonrotz.ch/wp-content/uploads/2017/08/SCCM-application-assets.png" alt="" width="1189" height="619" class="aligncenter size-full wp-image-4488" /></a>

The application package has PowerShell deployment type. Instead of an exectable a PowerShell is called to install the Application. Using a script allows to make additional configurations before and after the installation or uinstallation of the package.

In my next post I will share the script for deploying the created applications. Stay focused!