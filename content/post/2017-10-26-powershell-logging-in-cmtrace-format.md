---
id: 4627
title: 'PowerShell &#8211; Logging in CMTrace format'
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

[code lang="powershell"]
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
          [ValidateSet(&quot;Info&quot;, &quot;Warning&quot;, &quot;Error&quot;)]
          [String]$Type
    )

    switch ($Type) {
        &quot;Info&quot; { [int]$Type = 1 }
        &quot;Warning&quot; { [int]$Type = 2 }
        &quot;Error&quot; { [int]$Type = 3 }
    }

    # Create a log entry
    $Content = &quot;&lt;![LOG[$Message]LOG]!&gt;&quot; +`
        &quot;&lt;time=`&quot;$(Get-Date -Format &quot;HH:mm:ss.ffffff&quot;)`&quot; &quot; +`
        &quot;date=`&quot;$(Get-Date -Format &quot;M-d-yyyy&quot;)`&quot; &quot; +`
        &quot;component=`&quot;$Component`&quot; &quot; +`
        &quot;context=`&quot;$([System.Security.Principal.WindowsIdentity]::GetCurrent().Name)`&quot; &quot; +`
        &quot;type=`&quot;$Type`&quot; &quot; +`
        &quot;thread=`&quot;$([Threading.Thread]::CurrentThread.ManagedThreadId)`&quot; &quot; +`
        &quot;file=`&quot;`&quot;&gt;&quot;

    # Write the line to the log file
    Add-Content -Path $Path -Value $Content
}
[/code]

No  big matter, isn't it? We want to see it in action. Here is an example script that cycles its log history and writes an error entry whenever the script throws an error.

**Example.ps1**

[code lang="powershell"]
$LogCycle = 30
$LogFilePath = Join-Path $PSScriptRoot &quot;$(Get-Date -Format yyyy-M-dd) $($MyInvocation.MyCommand.Name).log&quot;

Write-Warning &quot;Delete log files older than $LogCycle days&quot;
Get-ChildItem -Path $PSScriptRoot | Where-Object {($Now - $_.LastWriteTime).Days -gt $LogCycle -and $_.extension -eq &quot;.log&quot;} | Remove-Item

try {

    throw &quot;Something failed.&quot;

} catch {

    Write-Error ($_ | Out-String)
    Write-Log -Path $LogFilePath -Message ($_ | Out-String) -Component $MyInvocation.MyCommand.Name -Type Error
}
[/code]

And finally the log entry opened in the CMTrace viewer:

<a href="https://janikvonrotz.ch/wp-content/uploads/2017/10/CMTrace-example-log.png"><img src="https://janikvonrotz.ch/wp-content/uploads/2017/10/CMTrace-example-log.png" alt="" width="665" height="453" class="aligncenter size-full wp-image-4628" /></a>