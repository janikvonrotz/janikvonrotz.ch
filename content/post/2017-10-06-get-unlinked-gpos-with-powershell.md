---
title: Get unlinked GPOs with PowerShell
date: 2017-10-06T10:06:49+00:00
author: Janik von Rotz
slug: get-unlinked-gpos-with-powershell
images:
  - /wp-content/uploads/2017/10/PowerShell-Get-unlinked-GPOs.png
categories:
  - scripting
tags:
  - compliance
  - group policy
  - powershell
  - reporting
---
In terms of IT compliance having valid GPOs is essential. They must be update to date and the GPO directory should be free of any unlinked GPOs. Retrieving a list of unlinked GPOs in the management console is impossible. With PowerShell it is quite easy.
<!--more-->
Take this function for example:

**Get-UnlinkedGPOs.ps1**

```powershell
function Get-UnlinkedGPOs {

    Import-Module GroupPolicy
    
    $Report = @() 
    $GPOs = Get-GPO -All
    $GPOs | ForEach-Object { 

        $GPO = $_

	    Write-Progress -Activity "Get GPO settings" -status "Analyze GPO: $($GPO.DisplayName)" -percentComplete ([int]([array]::IndexOf($GPOs, $GPO)/$GPOs.Count*100))
        
        $GPOReport = ([XML]$($GPO | Get-GPOReport -ReportType Xml)).GPO

        If(($GPOReport.LinksTo -eq $null) -or (-not ($GPOReport.LinksTo | Where-Object{$_.Enabled -eq $true}))){
            $Report += $GPO
        }
    }
     
    If ($Report.Count -eq 0) {
        Wirte-Warning "No unlinked GPOs found" 
    }else{ 
        return $Report
    }
}
```

Make sure the group policy PowerShell module is installed.

Once the function is available in your shell you can things like: `Get-UnlinkedGPOs | Select DisplayName, GpoStatus | Sort-Object DisplayName | Format-Table`