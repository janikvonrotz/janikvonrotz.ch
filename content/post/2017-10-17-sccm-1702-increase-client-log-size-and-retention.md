---
id: 4609
title: SCCM 1702 increase client log size and retention
date: 2017-10-17T13:46:41+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4609
permalink: /2017/10/17/sccm-1702-increase-client-log-size-and-retention/
specific_page_layout:
  - default-sidebar
image: /wp-content/uploads/2017/10/System-Center-Configuration-Manager-Logo.jpg
categories:
  - Configuration Manager
tags:
  - client
  - configuration manager
  - history
  - increase
  - logging
  - retention
  - size
  - system center
---
With additional steps in an image deployment task sequence the log files will grow quite big. By default the Configuration Manager client keeps a log history of 0 and a size limit of 2 MB for each log file. In result you'll find yourself missing important details when trying to debug a failed operating system deployment. At some point the log files will be cut off. In order to increase the log size and retention, parameters must be configured in two places.
<!--more-->

First we need to inject a SMSTS configuration file into the boot image to set the log parameters. To do so we use a simple PowerShell script.

**Copy-SMSTSConfigToBootImage.ps1**

```powershell
$BootImageFile = "C:\Program Files\Microsoft Configuration Manager\OSD\boot\x64\boot.wim"
$Index = "1"
$SMSIniFile = "D:\SCCM\Scripts\SMSTS.ini"

Write-Host "Start updating boot image: " (Get-Date).ToString()

Write-Host "Create temporary directory..."
$MountFolder = Join-Path $env:TEMP (Get-Random)
New-Item -ItemType Directory -Path $MountFolder | Out-Null
Write-Host "Created temporary directory: " $MountFolder

Write-Host "Mounting Windows-Image..." $BootImageFile
Mount-WindowsImage -Path $MountFolder -ImagePath $BootImageFile -Index $Index

Write-Host "Copy SMSTS.ini file..."
Copy-Item $SMSIniFile (Join-Path $MountFolder "Windows")

Write-Host "Dismount-WindowsImage..."
Dismount-WindowsImage -Path $MountFolder -Save -CheckIntegrity

Write-Host "Remove temporary directory..."
Remove-Item $MountFolder -Force

Write-Host "Complete upating boot image:" (Get-Date).ToString()
```

The script mounts a specified boot image and copies the `SMSTS.init` configuration file.

**SMSTS.ini**

```
[Logging]
LOGLEVEL=0
LOGMAXSIZE=5242880
LOGMAXHISTORY=3
DEBUGLOGGING=1
ENABLELOGGING=True
```

Once WinPE is unloaded and the selected windows image is being installed, we need set those parameters again. Only this time it is a bit easier.

Locate the **Setup Windows and Configuration Manager** step in your task sequence and set the installation properties as showed below below:

    CCMLOGMAXSIZE=5242880 CCMLOGLEVEL=0 CCMLOGMAXHISTORY=3

The CM client MSI will detect these parameters and write them to the registry at `HKLM:\SOFTWARE\Microsoft\CCM\Logging\@Global`.

If you want to make sure the SMSTS.ini file has been loaded before installing an OS, press F8 before selecting the task sequence (debug mode must be enabled). A command line pops up and you can navigate to `X:\Windows\` where the configuration file is stored.

I hope this tutorial works for you, otherwise ask me at any time.