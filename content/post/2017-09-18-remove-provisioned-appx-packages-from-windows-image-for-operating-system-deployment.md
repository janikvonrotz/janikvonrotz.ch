---
title: Remove provisioned appx packages from Windows image for operating system deployment
date: 2017-09-18T09:51:20+00:00
author: Janik von Rotz
slug: remove-provisioned-appx-packages-from-windows-image-for-operating-system-deployment
image: /wp-content/uploads/2017/09/Remove-selected-appx-packages-from-wim.png
categories:
  - Configuration Manager
tags:
  - apps
  - configuration manager
  - image
  - provisioned
  - remove
  - sccm
  - store
  - system center
  - windows
  - windows 10
---
While preparing a Windows image for SCCM deployment I looked for a viable solution to remove Windows apps from the image. SCCM offers a lot options  to execute this kind of action such as running a task sequence or install an application. But none of the options worked out for me. Either they were too complicated to configure or simply didn't work as expected. Today I've found a [script on TechNet](https://gallery.technet.microsoft.com/Removing-Built-in-apps-65dc387b) seemed to a good solution. This script showed me how easy it is to mount a windows image and remove the app packages directory from it. However, the script was outdated and didn't offer the option to remove only selected apps. That's why I've created my own remix of the script.
<!--more-->
Let's have a look:

**Remove-SelectedAppxPackagesFromWIM.ps1**

```powershell
$pathToWIM = "D:\SCCM\OSD\OSImages\Windows 10 Enterprise (x64).wim"
$index = "1"
$appList = @(
    "Windows.ContactSupport",
    "microsoft.windowscommunicationsapps",
    "Microsoft.SkypeApp",
    "Microsoft.ZuneVideo",
    "Microsoft.ZuneMusic",
    "Microsoft.XboxIdentityProvider",
    "Microsoft.XboxApp",
    "Microsoft.WindowsStore",
    "Microsoft.WindowsMaps",
    "Microsoft.StorePurchaseApp",
    "Microsoft.People",
    "Microsoft.OneConnect",
    "Microsoft.Office.OneNote",
    "Microsoft.MicrosoftSolitaireCollection",
    "Microsoft.MicrosoftOfficeHub",
    "Microsoft.Messaging",
    "Microsoft.Getstarted",
    "Microsoft.DesktopAppInstaller",
    "Microsoft.BingWeather",
    "Microsoft.3DBuilder"
)
<#
"Microsoft.MicrosoftEdge"
#>

Write-Host "Start:" (Get-Date).ToString()

Write-Host "Create temporary directory..."
$pathMountFolder = Join-Path $env:TEMP (Get-Random)
New-Item -ItemType Directory -Path $pathMountFolder

Write-Host "Mounting Windows-Image..." $pathToWIM
Mount-WindowsImage -Path $pathMountFolder -ImagePath $pathToWIM -Index $index

Write-Host "Remove the built-in apps:"
$Apps = Get-AppxProvisionedPackage -Path $pathMountFolder | ForEach-Object {

    if($appList -contains $_.displayName) {

        Write-Host "Delete:" $_.DisplayName
        Remove-AppxProvisionedPackage -Path $pathMountFolder -PackageName $_.PackageName

    } else {

        Write-Host $_.DisplayName "is not in app list or already removed"
    }    
}

Write-Host "Dismount-WindowsImage..."
Dismount-WindowsImage -Path $pathMountFolder  -Save -CheckIntegrity

Write-Host "Remove temporary directory..."
Remove-Item $pathMountFolder -Force

Write-Host "Complete:" (Get-Date).ToString()
```

First, the script creates a temporary folder and mounts the windows image. Next, app packages from a predefined list are removed. Finally, the image is dismounted and the temporary folder deleted.