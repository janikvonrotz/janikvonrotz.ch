---
title: Backup Active Directory with PowerShell
date: 2014-04-15T13:57:06+00:00
author: Janik von Rotz
slug: backup-active-directory-with-powershell
dsq_thread_id:
  - "2614387113"
images:
  - /wp-content/uploads/2013/12/PowerShell-and-ActiveDirectory.png
categories:
  - Active Directory
  - PowerShell
tags:
  - active
  - ad
  - backup
  - daily
  - directory
  - monthly
  - powershell
  - ps
  - restore
  - retention
  - weekly
---
Based on this [Technet article](http://technet.microsoft.com/en-us/library/dd581644(WS.10).aspx) I've developed a simple Active Directory backup tool with PowerShell.

What it does:

* Create a full snapshot from Active Directory
* Keep a daily, weekly and monthly snapshot
* Notify me if something failed (requires [PowerShell PowerUp](http://janikvonrotz.github.io/PowerShell-PowerUp/))
<!--more-->
```powershell
<#
$Metadata = @{
	Title = "Backup ActiveDirecotry"
	Filename = "Backup-ActiveDirectory.ps1"
	Description = ""
	Tags = "backup, active, directory, ntsutil"
	Project = ""
	Author = "Janik von Rotz"
	AuthorContact = "https://janikvonrotz.ch"
	CreateDate = "2014-04-15"
	LastEditDate = "2014-04-15"
	Url = ""
	Version = "0.0.0"
	License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/ch/ or 
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#>

try{

    <#
    #--------------------------------------------------#
    # about
    #--------------------------------------------------#
    
    The restore and backup process is described here: http://technet.microsoft.com/en-us/library/dd581644(WS.10).aspx
    
    #>
    
    #--------------------------------------------------#
    # settings
    #--------------------------------------------------#
    
    $Path = "C:\Powershell-PowerUp\backup\ActiveDirectory"
    
    #--------------------------------------------------#
    # main
    #--------------------------------------------------#
    
    # create backup file name
    $Filename = "ADBackupFull" + "#" + $((Get-Date -Format s) -replace ":","-") + ".bak"
    $Filepath = Join-Path $Path $Filename
    
    # backup active directory
    Invoke-Expression 'ntdsutil "activate instance ntds" ifm "create full $Filepath" quit quit'
    
    # get dates for backup retention exclusion
    $Today = Get-Date -Format d
    $FirstDateOfWeek = Get-Date (Get-Date).AddDays(-[int](Get-Date).Dayofweek) -Format d
    $FirstDateOfMonth = Get-Date -Day 1 -Format d
    
    # delete all backups except for today, first day of week and first day of month
    Get-ChildItem $Path | select *,@{L="CreationTimeDate";E={Get-Date $_.CreationTime -Format d}} | Group-Object CreationTimeDate | %{
        
        # only one backup per day
        if($_.Count -gt 1){
            
            $_.Group | Sort-Object CreationTime -Descending | Select-Object -Skip 1     
        }
                
        # keep only required backups
        $_.Group | Where-Object{$_.CreationTimeDate -ne $Today -and $_.CreationTimeDate -ne $FirstDateOfWeek -and $_.CreationTimeDate -ne $FirstDateOfMonth}
            
    } | Remove-Item -Recurse -Force

}catch{

    Write-PPErrorEventLog -Source "Backup ActiveDirectory" -ClearErrorVariable
}
```

Latest version of this script: [https://gist.github.com/10729700](https://gist.github.com/10729700)