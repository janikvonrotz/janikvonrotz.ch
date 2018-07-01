---
id: 4618
title: 'Configuration Manager - Configure requirement rules for deployment types with PowerShell'
date: 2017-10-20T09:49:46+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4618
permalink: /2017/10/20/configuration-manager-configure-requirement-rules-for-deployment-types-with-powershell/
image: /wp-content/uploads/2017/10/System-Center-Configuration-Manager-Logo.jpg
categories:
  - Configuration Manager
tags:
  - configuration manager
  - powershell
  - requirement
  - rules
  - scripting
  - system center
  - user device affinity
---
Configuration Manager applications can be equipped with powerful requirement rules. For example an application must be installed only if there is enough disk space on the target device or only if the device is the users primary device. The second example is an important requirement rule when working with [user device affinity](https://docs.microsoft.com/en-us/sccm/apps/deploy-use/link-users-and-devices-with-user-device-affinity). Configuring this kind of rule is done in a few seconds using the management console. However, scripting the rule with PowerShell is much more difficult. As of today the cmdlets provided by Microsoft for automating Configuration Manager assets do not support building requirement rules for deployment types. But as always there is a workaround. In my case I've decided to create an application template containing all requirement rules and copy specific rules from there to other applications.
<!--more-->

Once the template requirement rules are created, they can be copied using the script below.

**Copy-CMDeploymentTypeRule**
```ps
$SiteCode = "SITECODE"
$ProviderMachineName = "FQDN" 
if((Get-Module ConfigurationManager) -eq $null) {
    Import-Module "$($ENV:SMS_ADMIN_UI_PATH)\..\ConfigurationManager.psd1"
}
if((Get-PSDrive -Name $SiteCode -PSProvider CMSite -ErrorAction SilentlyContinue) -eq $null) {
    New-PSDrive -Name $SiteCode -PSProvider CMSite -Root $ProviderMachineName
}
Set-Location "$($SiteCode):\"

$RuleName = "Primary device Equals True"
# use * for all rules
$SourceApplicationName = "Application Requirement Rules Template"
$DestApplicationName = "VLC (2.2.6)"
$DestDeploymentTypeIndex = 0

# get the applications
$SourceApplication = Get-CMApplication -Name $SourceApplicationName | ConvertTo-CMApplication
$DestApplication = Get-CMApplication -Name $DestApplicationName | ConvertTo-CMApplication

# get requirement rules from source application
$Requirements = $SourceApplication.DeploymentTypes[0].Requirements | Where-Object {$_.Name -match $RuleName}

# apply requirement rules
$Requirements | ForEach-Object {
    
    $RuleExists = $DestApplication.DeploymentTypes[$DestDeploymentTypeIndex].Requirements | Where-Object {$_.Name -match $RuleName}
    if($RuleExists) {

        Write-Warning "The rule `"$($_.Name)`" already exists in target application deployment type"

    } else{
        
        Write-Host "Apply rule `"$($_.Name)`" on target application deployment type"

        # create new rule ID
        $_.RuleID = "Rule_$( [guid]::NewGuid())"

        $DestApplication.DeploymentTypes[$DestDeploymentTypeIndex].Requirements.Add($_)
    }
}

# push changes
$CMApplication = ConvertFrom-CMApplication -Application $DestApplication
$CMApplication.Put()
```

The `ConvertTo-CMApplication` cmdlet converts CM application objects to SDK objects. Using SDK objects offers more options to configure a CM application.

Source: [Technet - Deployment Type Requirement mit Powershell hinzufügen](https://social.technet.microsoft.com/Forums/de-DE/330ac938-6b8d-424a-a44e-1ad567739c54/deployment-type-requirement-mit-powershell-hinzufgen?forum=systemcenterde)