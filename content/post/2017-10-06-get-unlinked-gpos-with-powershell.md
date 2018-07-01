---
id: 4591
title: Get unlinked GPOs with PowerShell
date: 2017-10-06T10:06:49+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4591
permalink: /2017/10/06/get-unlinked-gpos-with-powershell/
image: /wp-content/uploads/2017/10/PowerShell-Get-unlinked-GPOs.png
categories:
  - PowerShell
tags:
  - compliance
  - group policy
  - list
  - management
  - powershell
  - report
  - settings
  - unlinked
---
In terms of IT compliance having valid GPOs is essential. They must be update to date and the GPO directory should be free of any unlinked GPOs. Retrieving a list of unlinked GPOs in the management console is impossible. With PowerShell it is quite easy.
<!--more-->
Take this function for example:

**Get-UnlinkedGPOs.ps1**

[code lang="powershell"]
function Get-UnlinkedGPOs {

    Import-Module GroupPolicy
    
    $Report = @() 
    $GPOs = Get-GPO -All
    $GPOs | ForEach-Object { 

        $GPO = $_

	    Write-Progress -Activity &quot;Get GPO settings&quot; -status &quot;Analyze GPO: $($GPO.DisplayName)&quot; -percentComplete ([int]([array]::IndexOf($GPOs, $GPO)/$GPOs.Count*100))
        
        $GPOReport = ([XML]$($GPO | Get-GPOReport -ReportType Xml)).GPO

        If(($GPOReport.LinksTo -eq $null) -or (-not ($GPOReport.LinksTo | Where-Object{$_.Enabled -eq $true}))){
            $Report += $GPO
        }
    }
     
    If ($Report.Count -eq 0) {
        Wirte-Warning &quot;No unlinked GPOs found&quot; 
    }else{ 
        return $Report
    }
}
[/code]

Make sure the group policy PowerShell module is installed.

Once the function is available in your shell you can things like: `Get-UnlinkedGPOs | Select DisplayName, GpoStatus | Sort-Object DisplayName | Format-Table`