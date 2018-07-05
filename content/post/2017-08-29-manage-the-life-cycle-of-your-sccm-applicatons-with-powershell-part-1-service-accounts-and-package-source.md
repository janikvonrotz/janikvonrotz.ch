---
id: 4475
title: 'Manage the life cycle of your SCCM applicatons with PowerShell - Part 1 Service Accounts and Package Source'
date: 2017-08-29T12:19:29+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4475
permalink: /2017/08/29/manage-the-life-cycle-of-your-sccm-applicatons-with-powershell-part-1-service-accounts-and-package-source/
image: /wp-content/uploads/2017/08/SCCM-PowerShell-e1503910243136.jpeg
categories:
  - Configuration Manager
tags:
  - "1702"
  - advanced
  - application
  - automation
  - configuration manager
  - console
  - current branch
  - deploy
  - deployment
  - installation
  - life cycle
  - management
  - powershell
  - sccm
  - script
  - system center configuration manager
  - wrapper
---
I'm currently planning and building a System Center Configuration Manager (SCCM) infrastructure for a local hospital. SCCM is a complex system composed of various components to make the client and software life cycle management feasible. While configuring SCCM is a tedious job which is done once, managing applications is a recurring process. That's why the application life cycle is the perfect candidate for scripted automation. With the most recent SCCM release Microsoft made it a lot easier to use the power of PowerShell. I've developed a few scripts which help creating, deploying and deleting SCCM applications. Configuring applications manually can be very bothersome and is always at risk for misconfiguration. The benefit for automation is huge.
<!--more-->

In my first post of this series I want to show you how I've scripted the service users and package source share for SCCM. It's not necessary that you run these scripts in your installation. I share them for a better understanding of the follow-up script. Furthermore, this posts intends to be a template for admins who are not sure how to configure their SCCM installation.

Separation of concerns is an important key principal of Microsoft products. That's why SCCM requires a lot of service users in order to run the system. Based on their guidelines I've written a script to create the necessary required accounts. Let's have a look:

**Create-CMServiceAccounts.ps1**

```powershell
Import-Module ActiveDirectory
Import-Module PowerUp

$OU = "OU=ServiceAccounts,OU=Admin,DC=example,DC=net"
$Domain = "example.net"
$Users = @(
    @{
        DisplayName = "SCCM Site"
        Name = "sccm_site"
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = "Runs the sccm primary site and its roles."
    }
     @{
        DisplayName = "SCCM Active Directory Discovery"
        Name = "sccm_discovery"
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = "Discovers users, groups and system in the Active Directory."
    }
    @{
        DisplayName = "SCCM Active Directory Forest"
        Name = "sccm_forest"
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = "Discovers Active Directory forests and publishes information in the system container."
    }
    @{
        DisplayName = "SCCM Proxy Access"
        Name = "sccm_proxy"
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = "Connects site roles with the proxy server."
    }
    @{
        DisplayName = "SCCM Capture OSI"
        Name = "sccm_captureosi"
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = "Stores the captured operating system images."
    }
    @{
        DisplayName = "SCCM Client Push"
        Name = "sccm_clientpush"
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = "Installs the sccm client on existing windows clients."
    }
    @{
        DisplayName = "SCCM Exchange Connection"
        Name = "sccm_exchange"
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = "Conencts the site with the Exchange server."
    }
    @{
        DisplayName = "SCCM Reporting Service"
        Name = "sccm_report"
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = "Runs the SCCM reporting service."
    }
    @{
        DisplayName = "SCCM Domain Join"
        Name = "sccm_domainjoin"
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = "Joins systems during the installation to the domain."
    }
    @{
        DisplayName = "SCCM SQL Service"
        Name = "sccm_sql"
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = "Runs the SQL server."
    }
    @{
        DisplayName = "SCCM Network Access"
        Name = "sccm_network"
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = "Accesses the packages source and loads files required for installation and deployment."
    }
    @{
        DisplayName = "SCCM WSUS"
        Name = "sccm_wsus"
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = "Connects the site with the WSUS server."
    }
)

$Users | ForEach-Object {
    Write-Host "Create AD Service Account: $($_.Name) with Password: $($_.Password)"
    New-ADUser -Name $_.DisplayName -DisplayName $_.DisplayName -AccountPassword (ConvertTo-SecureString -AsPlainText $_.Password -Force) -Enabled $true -Path $OU -CannotChangePassword $_.CannotChangePassword -ChangePasswordAtLogon $_.ChangePasswordAtLogon -PasswordNeverExpires $_.PasswordNeverExpires -SamAccountName $_.Name -UserPrincipalName "$($_.Name)@$Domain" -Description $_.Description
}
```

As you can see the script requires two PowerShell modules. One ist the default Active Directory module and the other is the `PowerUp` module, which has been published by me. The PowerUp module is only required for the `Get-RandomPassword` function and can easily be ditched for something else, such as a static password list.

These service accounts must be configured in the SCCM console and will be granted different access rights on the package source share. The package source share is the folder structure where you store all the files required by SCCM to deploy operating systems and applications. Let's check out the script to build the folder structure:

**Create-CMPackageSourceShare.ps1**

```powershell
# set the parameters
$Source = "D:\SCCM"
$ShareName = "PackageSource"
$NetworkAccount = "go4ksow.net\sccm_network"
$SiteAccount = "go4ksow.net\sccm_site"
$SystemAccount = "NT AUTHORITY\SYSTEM"
$LocalServiceAccount = "LOCALSERVICE"
$CaputreOSIAccount = "go4ksow.net\sccm_captureosi"

# create the source directory
New-Item -ItemType Directory -Path "$Source"

# Application
New-Item -ItemType Directory -Path "$Source\Applications"
New-Item -ItemType Directory -Path "$Source\Applications\Adobe"
New-Item -ItemType Directory -Path "$Source\Applications\Citrix"
New-Item -ItemType Directory -Path "$Source\Applications\Microsoft"
New-Item -ItemType Directory -Path "$Source\Applications\Microsoft\Office 2016"

# App-V
New-Item -ItemType Directory -Path "$Source\App-V"
New-Item -ItemType Directory -Path "$Source\App-V\Packages"
New-Item -ItemType Directory -Path "$Source\App-V\Source"

# Device Applications
New-Item -ItemType Directory -Path "$Source\DeviceApplications"
New-Item -ItemType Directory -Path "$Source\DeviceApplications\HP"
New-Item -ItemType Directory -Path "$Source\DeviceApplications\HP\Probook 450"
New-Item -ItemType Directory -Path "$Source\DeviceApplications\HP\Probook 450\x86"
New-Item -ItemType Directory -Path "$Source\DeviceApplications\HP\Probook 450\x64"

# Import
New-Item -ItemType Directory -Path "$Source\Import"
New-Item -ItemType Directory -Path "$Source\Import\Baselines"
New-Item -ItemType Directory -Path "$Source\Import\MOFs"
New-Item -ItemType Directory -Path "$Source\Import\Task Sequences"

# Logs
New-Item -ItemType Directory -Path "$Source\Logs"
New-Item -ItemType Directory -Path "$Source\Logs\MDTLogs"
New-Item -ItemType Directory -Path "$Source\Logs\MDTLogsDL"

# Operating System Deployment
New-Item -ItemType Directory -Path "$Source\OSD"
New-Item -ItemType Directory -Path "$Source\OSD\BootImages"
New-Item -ItemType Directory -Path "$Source\OSD\Captures"
New-Item -ItemType Directory -Path "$Source\OSD\MDTSettings"
New-Item -ItemType Directory -Path "$Source\OSD\MDTToolkit"
New-Item -ItemType Directory -Path "$Source\OSD\OSImages"
New-Item -ItemType Directory -Path "$Source\OSD\OSInstall"
New-Item -ItemType Directory -Path "$Source\OSD\Prestart"
New-Item -ItemType Directory -Path "$Source\OSD\USMT"
New-Item -ItemType Directory -Path "$Source\OSD\DriverPackages"
New-Item -ItemType Directory -Path "$Source\OSD\DriverPackages"
New-Item -ItemType Directory -Path "$Source\OSD\DriverPackages\HP"
New-Item -ItemType Directory -Path "$Source\OSD\DriverPackages\HP\Probook 450"
New-Item -ItemType Directory -Path "$Source\OSD\DriverSources"
New-Item -ItemType Directory -Path "$Source\OSD\DriverSources"
New-Item -ItemType Directory -Path "$Source\OSD\DriverSources\HP"
New-Item -ItemType Directory -Path "$Source\OSD\DriverSources\HP\Probook 450"
New-Item -ItemType Directory -Path "$Source\OSD\DriverSources\HP\Probook 450\Realtek Ethernet Controller Drivers"

# Script
New-Item -ItemType Directory -Path "$Source\Scripts"

# Tools
New-Item -ItemType Directory -Path "$Source\Tools"
New-Item -ItemType Directory -Path "$Source\Tools\PSTools"

# Windows Updates
New-Item -ItemType Directory -Path "$Source\WindowsUpdates"
New-Item -ItemType Directory -Path "$Source\WindowsUpdates\Office 2016"
New-Item -ItemType Directory -Path "$Source\WindowsUpdates\Windows 10"

# WSUS
New-Item -ItemType Directory -Path "$Source\WSUS"

# create the share
New-SmbShare -Name "$ShareName" -Path "$Source" -CachingMode None -ReadAccess Everyone -FullAccess $SiteAccount, $SystemAccount
# Get-SmbShare | where{$_.Name -eq "PackageSource"} | Remove-SmbShare

# set security permissions
$Acl = Get-Acl "$Source\Logs"
$Ar = New-Object System.Security.AccessControl.FileSystemAccessRule($NetworkAccount,"FullControl", "ContainerInherit, ObjectInherit", "None", "Allow")
$Acl.SetAccessRule($Ar)
Set-Acl "$Source\Logs" $Acl

$Acl = Get-Acl "$Source\OSD\Captures"
$Ar = New-Object System.Security.AccessControl.FileSystemAccessRule($CaputreOSIAccount,"FullControl", "ContainerInherit, ObjectInherit", "None", "Allow")
$Acl.SetAccessRule($Ar)
Set-Acl "$Source\OSD\Captures" $Acl

$Acl = Get-Acl "$Source\StateCapture"
$Ar = New-Object System.Security.AccessControl.FileSystemAccessRule($LocalServiceAccount,"FullControl", "ContainerInherit, ObjectInherit", "None", "Allow")
$Acl.SetAccessRule($Ar)
Set-Acl "$Source\StateCapture" $Acl
```

Of course you do not need all of these folders. Adjust the script according to your requirements and preferences.

Here's a short explanation for each folder:

**Application**  
This where the application installation files are stored.  
`Applications\[Vendor]\[Product]\[Version]`

**App-V**  
Source folder for App-V applications.  
`App-V\[Packages,Source]`

**Device Applications**  
Vendor specific applicatios, that are applied when deploying an image.  
`DeviceApplications\[Vendor]\[Product]\[Architecture]`

**Import**  
Going to import data into the SCCM database? Put them here.  
`Import\[Type]`

**Logs**  
Store important SCCM log files here.  
`Logs\[Type]`

**Operating System Deployment**  
Everything related for deploying os images.  
`OSD\[BootImages,Captures,MDTSettings,MDTToolkit,OSImages,OSInstall,Prestart]`  
Put driver files and source packages into these folders.  
`OSD\[DriverPackages,DriverSources]\[Vendor]\[Product and Version]\[Driver Type]`  

**Script**  
PowerShell scripts for SCCM configurations.  
`Script`

**Tools**  
Any SCCM related tool.  
`Tools\[Type]`

**Windows Updates**  
Source folder for the software update deployment packages.  
`WindowsUpdates\[Product]`

**WSUS**  
This is the WSUS data folder.  
`WSUS`

In the next post I will share the script to create new applications. Stay tuned!