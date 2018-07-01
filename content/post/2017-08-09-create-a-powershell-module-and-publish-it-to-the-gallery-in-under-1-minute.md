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

[code lang="powershell"]
# Go to the modules directory

Set-Location &quot;C:\Windows\system32\WindowsPowerShell\v1.0\Modules\&quot;

# Create the module folder

if (!(Test-Path &quot;./PowerUp&quot;)) {
    New-Item -ItemType Directory -Path &quot;./PowerUp&quot;
}

# Create the module

@'
function Get-RandomPassword {
    
    $numbers = 1..9
    $consonants = &quot;b&quot;,&quot;c&quot;,&quot;d&quot;,&quot;f&quot;,&quot;g&quot;,&quot;h&quot;,&quot;k&quot;,&quot;l&quot;,&quot;m&quot;,&quot;n&quot;,&quot;p&quot;,&quot;r&quot;,&quot;s&quot;,&quot;t&quot;,&quot;v&quot;,&quot;w&quot;,&quot;x&quot;,&quot;z&quot;
    $nopeletters = &quot;j&quot;,&quot;q&quot;,&quot;y&quot;
    $vocals = &quot;a&quot;,&quot;e&quot;,&quot;i&quot;,&quot;o&quot;,&quot;u&quot;
    $dotsandstuff = &quot;,&quot;,&quot;.&quot;,&quot;-&quot;
    $nopedotsandstuff = &quot;;&quot;,&quot;:&quot;,&quot;_&quot;

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
'@ | Set-Content -Path &quot;./PowerUp/PowerUp.psm1&quot;

# Create the module manifest

New-ModuleManifest &quot;./PowerUp/PowerUp.psd1&quot; -RootModule &quot;PowerUp&quot; -FunctionsToExport Get-RandomPassword -ModuleVersion &quot;1.0.0&quot; -Author &quot;Janik von Rotz&quot; -Description &quot;A collection of useful PowerShell functions.&quot;

# Test the module manifest

Test-ModuleManifest .\PowerUp\PowerUp.psd1

# Create an account at https://www.powershellgallery.com and copy the api key from your profile settings.

# Publish the module to the gallery

Publish-Module -Name PowerUp -NuGetApiKey API_KEY
[/code]

**InstallAndImport-PowerShellModule.ps1**

[code lang="powershell"]
# On another computer install the module from the gallery

Install-Module -Name PowerUp

# Import the module

Import-Module PowerUp

# Run the module function

Get-RandomPassword
[/code]