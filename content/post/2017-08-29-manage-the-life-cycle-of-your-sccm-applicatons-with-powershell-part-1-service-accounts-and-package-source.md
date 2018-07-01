---
id: 4475
title: 'Manage the life cycle of your SCCM applicatons with PowerShell &#8211; Part 1 Service Accounts and Package Source'
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

[code lang="powershell"]
Import-Module ActiveDirectory
Import-Module PowerUp

$OU = &quot;OU=ServiceAccounts,OU=Admin,DC=example,DC=net&quot;
$Domain = &quot;example.net&quot;
$Users = @(
    @{
        DisplayName = &quot;SCCM Site&quot;
        Name = &quot;sccm_site&quot;
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = &quot;Runs the sccm primary site and its roles.&quot;
    }
     @{
        DisplayName = &quot;SCCM Active Directory Discovery&quot;
        Name = &quot;sccm_discovery&quot;
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = &quot;Discovers users, groups and system in the Active Directory.&quot;
    }
    @{
        DisplayName = &quot;SCCM Active Directory Forest&quot;
        Name = &quot;sccm_forest&quot;
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = &quot;Discovers Active Directory forests and publishes information in the system container.&quot;
    }
    @{
        DisplayName = &quot;SCCM Proxy Access&quot;
        Name = &quot;sccm_proxy&quot;
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = &quot;Connects site roles with the proxy server.&quot;
    }
    @{
        DisplayName = &quot;SCCM Capture OSI&quot;
        Name = &quot;sccm_captureosi&quot;
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = &quot;Stores the captured operating system images.&quot;
    }
    @{
        DisplayName = &quot;SCCM Client Push&quot;
        Name = &quot;sccm_clientpush&quot;
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = &quot;Installs the sccm client on existing windows clients.&quot;
    }
    @{
        DisplayName = &quot;SCCM Exchange Connection&quot;
        Name = &quot;sccm_exchange&quot;
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = &quot;Conencts the site with the Exchange server.&quot;
    }
    @{
        DisplayName = &quot;SCCM Reporting Service&quot;
        Name = &quot;sccm_report&quot;
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = &quot;Runs the SCCM reporting service.&quot;
    }
    @{
        DisplayName = &quot;SCCM Domain Join&quot;
        Name = &quot;sccm_domainjoin&quot;
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = &quot;Joins systems during the installation to the domain.&quot;
    }
    @{
        DisplayName = &quot;SCCM SQL Service&quot;
        Name = &quot;sccm_sql&quot;
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = &quot;Runs the SQL server.&quot;
    }
    @{
        DisplayName = &quot;SCCM Network Access&quot;
        Name = &quot;sccm_network&quot;
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = &quot;Accesses the packages source and loads files required for installation and deployment.&quot;
    }
    @{
        DisplayName = &quot;SCCM WSUS&quot;
        Name = &quot;sccm_wsus&quot;
        Password = Get-RandomPassword
        CannotChangePassword = $true
        ChangePasswordAtLogon = $false
        PasswordNeverExpires = $true
        Description = &quot;Connects the site with the WSUS server.&quot;
    }
)

$Users | ForEach-Object {
    Write-Host &quot;Create AD Service Account: $($_.Name) with Password: $($_.Password)&quot;
    New-ADUser -Name $_.DisplayName -DisplayName $_.DisplayName -AccountPassword (ConvertTo-SecureString -AsPlainText $_.Password -Force) -Enabled $true -Path $OU -CannotChangePassword $_.CannotChangePassword -ChangePasswordAtLogon $_.ChangePasswordAtLogon -PasswordNeverExpires $_.PasswordNeverExpires -SamAccountName $_.Name -UserPrincipalName &quot;$($_.Name)@$Domain&quot; -Description $_.Description
}
[/code]

As you can see the script requires two PowerShell modules. One ist the default Active Directory module and the other is the `PowerUp` module, which has been published by me. The PowerUp module is only required for the `Get-RandomPassword` function and can easily be ditched for something else, such as a static password list.

These service accounts must be configured in the SCCM console and will be granted different access rights on the package source share. The package source share is the folder structure where you store all the files required by SCCM to deploy operating systems and applications. Let's check out the script to build the folder structure:

**Create-CMPackageSourceShare.ps1**

[code lang="powershell"]
# set the parameters
$Source = &quot;D:\SCCM&quot;
$ShareName = &quot;PackageSource&quot;
$NetworkAccount = &quot;go4ksow.net\sccm_network&quot;
$SiteAccount = &quot;go4ksow.net\sccm_site&quot;
$SystemAccount = &quot;NT AUTHORITY\SYSTEM&quot;
$LocalServiceAccount = &quot;LOCALSERVICE&quot;
$CaputreOSIAccount = &quot;go4ksow.net\sccm_captureosi&quot;

# create the source directory
New-Item -ItemType Directory -Path &quot;$Source&quot;

# Application
New-Item -ItemType Directory -Path &quot;$Source\Applications&quot;
New-Item -ItemType Directory -Path &quot;$Source\Applications\Adobe&quot;
New-Item -ItemType Directory -Path &quot;$Source\Applications\Citrix&quot;
New-Item -ItemType Directory -Path &quot;$Source\Applications\Microsoft&quot;
New-Item -ItemType Directory -Path &quot;$Source\Applications\Microsoft\Office 2016&quot;

# App-V
New-Item -ItemType Directory -Path &quot;$Source\App-V&quot;
New-Item -ItemType Directory -Path &quot;$Source\App-V\Packages&quot;
New-Item -ItemType Directory -Path &quot;$Source\App-V\Source&quot;

# Device Applications
New-Item -ItemType Directory -Path &quot;$Source\DeviceApplications&quot;
New-Item -ItemType Directory -Path &quot;$Source\DeviceApplications\HP&quot;
New-Item -ItemType Directory -Path &quot;$Source\DeviceApplications\HP\Probook 450&quot;
New-Item -ItemType Directory -Path &quot;$Source\DeviceApplications\HP\Probook 450\x86&quot;
New-Item -ItemType Directory -Path &quot;$Source\DeviceApplications\HP\Probook 450\x64&quot;

# Import
New-Item -ItemType Directory -Path &quot;$Source\Import&quot;
New-Item -ItemType Directory -Path &quot;$Source\Import\Baselines&quot;
New-Item -ItemType Directory -Path &quot;$Source\Import\MOFs&quot;
New-Item -ItemType Directory -Path &quot;$Source\Import\Task Sequences&quot;

# Logs
New-Item -ItemType Directory -Path &quot;$Source\Logs&quot;
New-Item -ItemType Directory -Path &quot;$Source\Logs\MDTLogs&quot;
New-Item -ItemType Directory -Path &quot;$Source\Logs\MDTLogsDL&quot;

# Operating System Deployment
New-Item -ItemType Directory -Path &quot;$Source\OSD&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\BootImages&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\Captures&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\MDTSettings&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\MDTToolkit&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\OSImages&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\OSInstall&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\Prestart&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\USMT&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\DriverPackages&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\DriverPackages&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\DriverPackages\HP&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\DriverPackages\HP\Probook 450&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\DriverSources&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\DriverSources&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\DriverSources\HP&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\DriverSources\HP\Probook 450&quot;
New-Item -ItemType Directory -Path &quot;$Source\OSD\DriverSources\HP\Probook 450\Realtek Ethernet Controller Drivers&quot;

# Script
New-Item -ItemType Directory -Path &quot;$Source\Scripts&quot;

# Tools
New-Item -ItemType Directory -Path &quot;$Source\Tools&quot;
New-Item -ItemType Directory -Path &quot;$Source\Tools\PSTools&quot;

# Windows Updates
New-Item -ItemType Directory -Path &quot;$Source\WindowsUpdates&quot;
New-Item -ItemType Directory -Path &quot;$Source\WindowsUpdates\Office 2016&quot;
New-Item -ItemType Directory -Path &quot;$Source\WindowsUpdates\Windows 10&quot;

# WSUS
New-Item -ItemType Directory -Path &quot;$Source\WSUS&quot;

# create the share
New-SmbShare -Name &quot;$ShareName&quot; -Path &quot;$Source&quot; -CachingMode None -ReadAccess Everyone -FullAccess $SiteAccount, $SystemAccount
# Get-SmbShare | where{$_.Name -eq &quot;PackageSource&quot;} | Remove-SmbShare

# set security permissions
$Acl = Get-Acl &quot;$Source\Logs&quot;
$Ar = New-Object System.Security.AccessControl.FileSystemAccessRule($NetworkAccount,&quot;FullControl&quot;, &quot;ContainerInherit, ObjectInherit&quot;, &quot;None&quot;, &quot;Allow&quot;)
$Acl.SetAccessRule($Ar)
Set-Acl &quot;$Source\Logs&quot; $Acl

$Acl = Get-Acl &quot;$Source\OSD\Captures&quot;
$Ar = New-Object System.Security.AccessControl.FileSystemAccessRule($CaputreOSIAccount,&quot;FullControl&quot;, &quot;ContainerInherit, ObjectInherit&quot;, &quot;None&quot;, &quot;Allow&quot;)
$Acl.SetAccessRule($Ar)
Set-Acl &quot;$Source\OSD\Captures&quot; $Acl

$Acl = Get-Acl &quot;$Source\StateCapture&quot;
$Ar = New-Object System.Security.AccessControl.FileSystemAccessRule($LocalServiceAccount,&quot;FullControl&quot;, &quot;ContainerInherit, ObjectInherit&quot;, &quot;None&quot;, &quot;Allow&quot;)
$Acl.SetAccessRule($Ar)
Set-Acl &quot;$Source\StateCapture&quot; $Acl
[/code]

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