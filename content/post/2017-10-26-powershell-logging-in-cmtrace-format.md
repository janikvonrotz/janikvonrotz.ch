---
id: 4627
title: 'PowerShell - Logging in CMTrace format'
date: 2017-10-26T07:15:24+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4627
permalink: /2017/10/26/powershell-logging-in-cmtrace-format/
image: /wp-content/uploads/2017/10/CMTrace-tool.png
categories:
  - Configuration Manager
  - PowerShell
tags:
  - cmtrace
  - configuration manager
  - cycle
  - format
  - logging
  - powershell
  - scripting
  - system center
---
CMTrace is probably the first choice for a log viewer in a Microsoft environment. When working with System Center Configuration Manager there aren't any alternatives available. In a recent scenario I had to write log files in the CMtrace format. There are already many cmdlets available to do so, however, most of them did not work well or were overengineered. There I've taken a look at the CMTrace format specs and wrote a PowerShell function to create compatible log files.
<!--more-->

**Write-Log.ps1**

```powershell
function Write-log {

    [CmdletBinding()]
    Param(
          [parameter(Mandatory=$true)]
          [String]$Path,

          [parameter(Mandatory=$true)]
          [String]$Message,

          [parameter(Mandatory=$true)]
          [String]$Component,

          [Parameter(Mandatory=$true)]
          [ValidateSet("Info", "Warning", "Error")]
          [String]$Type
    )

    switch ($Type) {
        "Info" { [int]$Type = 1 }
        "Warning" { [int]$Type = 2 }
        "Error" { [int]$Type = 3 }
    }

    # Create a log entry
    $Content = "<![LOG[$Message]LOG]!>" +`
        "<time=`"$(Get-Date -Format "HH:mm:ss.ffffff")`" " +`
        "date=`"$(Get-Date -Format "M-d-yyyy")`" " +`
        "component=`"$Component`" " +`
        "context=`"$([System.Security.Principal.WindowsIdentity]::GetCurrent().Name)`" " +`
        "type=`"$Type`" " +`
        "thread=`"$([Threading.Thread]::CurrentThread.ManagedThreadId)`" " +`
        "file=`"`">"

    # Write the line to the log file
    Add-Content -Path $Path -Value $Content
}
```

No  big matter, isn't it? We want to see it in action. Here is an example script that cycles its log history and writes an error entry whenever the script throws an error.

**Example.ps1**

```powershell
$LogCycle = 30
$LogFilePath = Join-Path $PSScriptRoot "$(Get-Date -Format yyyy-M-dd) $($MyInvocation.MyCommand.Name).log"

Write-Warning "Delete log files older than $LogCycle days"
Get-ChildItem -Path $PSScriptRoot | Where-Object {($Now - $_.LastWriteTime).Days -gt $LogCycle -and $_.extension -eq ".log"} | Remove-Item

try {

    throw "Something failed."

} catch {

    Write-Error ($_ | Out-String)
    Write-Log -Path $LogFilePath -Message ($_ | Out-String) -Component $MyInvocation.MyCommand.Name -Type Error
}
```

And finally the log entry opened in the CMTrace viewer:

[![Untitled](https://janikvonrotz.ch/wp-content/uploads/2017/10/CMTrace-example-log.png)](https://janikvonrotz.ch/wp-content/uploads/2017/10/CMTrace-example-log.png)