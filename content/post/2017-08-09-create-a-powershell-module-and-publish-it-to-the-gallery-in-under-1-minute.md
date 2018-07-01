---
id: 4418
title: Create a PowerShell module and publish it to the Gallery in under 1 minute
date: 2017-08-09T11:26:58+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4418
permalink: /2017/08/09/create-a-powershell-module-and-publish-it-to-the-gallery-in-under-1-minute/
image: /wp-content/uploads/2015/06/PowerShell-logo-e1433137513315.png
categories:
  - PowerShell
tags:
  - create
  - deploy
  - example
  - gallery
  - install
  - manifest
  - module
  - powershell
  - publish
---
This tutorial is basically a script that creates a PowerShell module and publishes it to the PowerShell Gallery. Another scripts tells you how to install the published module and make us of it.
<!--more-->

Make sure that you either have PowerShell 5.0 installed or the PowerShellGet module installed and imported.

**CreateAndPublish-PowerShellModule.ps1**

```ps
# Go to the modules directory

Set-Location "C:\Windows\system32\WindowsPowerShell\v1.0\Modules\"

# Create the module folder

if (!(Test-Path "./PowerUp")) {
    New-Item -ItemType Directory -Path "./PowerUp"
}

# Create the module

@'
function Get-RandomPassword {
    
    $numbers = 1..9
    $consonants = "b","c","d","f","g","h","k","l","m","n","p","r","s","t","v","w","x","z"
    $nopeletters = "j","q","y"
    $vocals = "a","e","i","o","u"
    $dotsandstuff = ",",".","-"
    $nopedotsandstuff = ";",":","_"

    return (Get-Random $consonants).ToString().ToUpper() + 
    (Get-Random $vocals) + 
    (Get-Random $consonants) + 
    (Get-Random $vocals) + 
    (Get-Random $consonants) + 
    (Get-Random $vocals) + 
    (Get-Random $numbers) +  
    (Get-Random $numbers) + 
    (Get-Random $numbers) + 
    (Get-Random $dotsandstuff)
}
'@ | Set-Content -Path "./PowerUp/PowerUp.psm1"

# Create the module manifest

New-ModuleManifest "./PowerUp/PowerUp.psd1" -RootModule "PowerUp" -FunctionsToExport Get-RandomPassword -ModuleVersion "1.0.0" -Author "Janik von Rotz" -Description "A collection of useful PowerShell functions."

# Test the module manifest

Test-ModuleManifest .\PowerUp\PowerUp.psd1

# Create an account at https://www.powershellgallery.com and copy the api key from your profile settings.

# Publish the module to the gallery

Publish-Module -Name PowerUp -NuGetApiKey API_KEY
```

**InstallAndImport-PowerShellModule.ps1**

```ps
# On another computer install the module from the gallery

Install-Module -Name PowerUp

# Import the module

Import-Module PowerUp

# Run the module function

Get-RandomPassword
```