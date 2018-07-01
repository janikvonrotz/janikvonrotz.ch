---
id: 4564
title: Remove provisioned appx packages from Windows image for operating system deployment
date: 2017-09-18T09:51:20+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4564
permalink: /2017/09/18/remove-provisioned-appx-packages-from-windows-image-for-operating-system-deployment/
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

[code lang="powershell"]
$pathToWIM = &quot;D:\SCCM\OSD\OSImages\Windows 10 Enterprise (x64).wim&quot;
$index = &quot;1&quot;
$appList = @(
    &quot;Windows.ContactSupport&quot;,
    &quot;microsoft.windowscommunicationsapps&quot;,
    &quot;Microsoft.SkypeApp&quot;,
    &quot;Microsoft.ZuneVideo&quot;,
    &quot;Microsoft.ZuneMusic&quot;,
    &quot;Microsoft.XboxIdentityProvider&quot;,
    &quot;Microsoft.XboxApp&quot;,
    &quot;Microsoft.WindowsStore&quot;,
    &quot;Microsoft.WindowsMaps&quot;,
    &quot;Microsoft.StorePurchaseApp&quot;,
    &quot;Microsoft.People&quot;,
    &quot;Microsoft.OneConnect&quot;,
    &quot;Microsoft.Office.OneNote&quot;,
    &quot;Microsoft.MicrosoftSolitaireCollection&quot;,
    &quot;Microsoft.MicrosoftOfficeHub&quot;,
    &quot;Microsoft.Messaging&quot;,
    &quot;Microsoft.Getstarted&quot;,
    &quot;Microsoft.DesktopAppInstaller&quot;,
    &quot;Microsoft.BingWeather&quot;,
    &quot;Microsoft.3DBuilder&quot;
)
&lt;#
&quot;Microsoft.MicrosoftEdge&quot;
#&gt;

Write-Host &quot;Start:&quot; (Get-Date).ToString()

Write-Host &quot;Create temporary directory...&quot;
$pathMountFolder = Join-Path $env:TEMP (Get-Random)
New-Item -ItemType Directory -Path $pathMountFolder

Write-Host &quot;Mounting Windows-Image...&quot; $pathToWIM
Mount-WindowsImage -Path $pathMountFolder -ImagePath $pathToWIM -Index $index

Write-Host &quot;Remove the built-in apps:&quot;
$Apps = Get-AppxProvisionedPackage -Path $pathMountFolder | ForEach-Object {

    if($appList -contains $_.displayName) {

        Write-Host &quot;Delete:&quot; $_.DisplayName
        Remove-AppxProvisionedPackage -Path $pathMountFolder -PackageName $_.PackageName

    } else {

        Write-Host $_.DisplayName &quot;is not in app list or already removed&quot;
    }    
}

Write-Host &quot;Dismount-WindowsImage...&quot;
Dismount-WindowsImage -Path $pathMountFolder  -Save -CheckIntegrity

Write-Host &quot;Remove temporary directory...&quot;
Remove-Item $pathMountFolder -Force

Write-Host &quot;Complete:&quot; (Get-Date).ToString()
[/code]

First, the script creates a temporary folder and mounts the windows image. Next, app packages from a predefined list are removed. Finally, the image is dismounted and the temporary folder deleted.