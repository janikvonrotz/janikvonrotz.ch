---
id: 2053
title: Backup Active Directory Group Policies with PowerShell
date: 2014-04-22T08:02:13+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=2053
permalink: /2014/04/22/backup-active-directory-group-policies-with-powershell/
dsq_thread_id:
  - "2629693584"
image: /wp-content/uploads/2013/10/Group-Policy.jpg
categories:
  - Active Directory
  - PowerShell
tags:
  - backup
  - full
  - group
  - objects
  - policy
  - powershell
  - snaphost
---
Based on my last [Active Directory backup script](https://janikvonrotz.ch/2014/04/15/backup-active-directory-with-powershell/) I've developed a similar script to backup all group policies.

What it does:

* Create a daily full snapshot of all group policy objects
* Keep a daily, weekly and monthly snapshot
* Notify me if something failed (requires PowerShell PowerUp)
<!--more-->
[code lang="ps"]
<#
$Metadata = @{
    Title = "Backup Active Directory Group Policies"
    Filename = "Backup-ADGroupPolicies.ps1"
    Description = ""
    Tags = "backup, active, directory, group, object, policy"
    Project = ""
    Author = "Janik von Rotz"
    AuthorContact = "http://janikvonrotz.ch"
    CreateDate = "2014-04-22"
    LastEditDate = "2014-04-22
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

    #--------------------------------------------------#
    # modules
    #--------------------------------------------------#
    
    Import-Module ActiveDirectory
    Import-Module GroupPolicy
    
    #--------------------------------------------------#
    # settings
    #--------------------------------------------------#

    $Path = "C:\backup\GroupPolicy"

    #--------------------------------------------------#
    # main
    #--------------------------------------------------#

    # create backup file name
    $Filename = "GroupPolicyFull" + "#" + $((Get-Date -Format s) -replace ":","-") + ".bak"
    $Filepath = Join-Path $Path $Filename

    # backup active directory
    Get-GPO -All | ForEach-Object{
    
        $GPOFilepath = Join-Path $Filepath $_.DisplayName    
        New-Item -Path $GPOFilepath -ItemType Directory
        Backup-GPO -Guid $_.ID -Path $GPOFilepath    
    }
    
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

    Write-PPErrorEventLog -Source "Backup ActiveDirectory Group Policies" -ClearErrorVariable
}
[/code]

Latest version of this script: [https://gist.github.com/11167763](https://gist.github.com/11167763)