---
title: Forward Windows event log entries to syslog server
date: 2017-11-02T12:41:37+00:00
author: Janik von Rotz
slug: forward-windows-event-log-entries-to-syslog-server
images:
  - /wp-content/uploads/2017/08/SCCM-PowerShell-e1503910243136.jpeg
categories:
  - PowerShell
  - Windows Server
tags:
  - event log
  - forward
  - powershell
  - schedule
  - scripting
  - syslog
  - windows
---
Syslog is the defacto standard for sending log messages in an IP network. Instead of pulling log messages from a remote computer as you would do it in a windows environment, the log files are sent by remote computers to a central log repository. This way of managing log files has become the standard for Linux / Unix environments. As our IT systems tends to become hybrids, the questions arises how it possible to send syslog messages from a windows computer. In this post I will present you a simple approach.
<!--more-->

In the scenario we want to forward all security related messages from the windows log to the syslog server using PowerShell. To do so 3 script files are required.

First we need a function to send the message. Use [PowerShell Gallery - Send-SyslogMessage](https://www.powershellgallery.com/packages/Posh-SYSLOG/2.0.3/Content/Functions%5CSend-SyslogMessage.ps1) function from the Posh-SYSLOG module.

Second a function is required for writing local log files. This one is optional, but recommended. The [Write-Log](https://janikvonrotz.ch/2017/10/26/powershell-logging-in-cmtrace-format) cmdlet has been presented in a [previous post](https://janikvonrotz.ch/2017/10/26/powershell-logging-in-cmtrace-format).

Third and last the script that is forwarding the windows log messages to our syslog server.

**Send-SecurityEventsToPRTG.ps1**

```powershell
param(
    [String]$ServerName = "SYSLOG_HOSTNAME"
)

Import-Module (Join-Path $PSScriptRoot "Send-SyslogMessage.ps1")
Import-Module (Join-Path $PSScriptRoot "Write-Log.ps1")

$LogName = "security"
$EndTime = Get-Date
$StartTime = $EndTime.AddHours(-1)
$LogCycle = 10
$LogFilePath = Join-Path $PSScriptRoot "$(Get-Date -Format yyyy-M-dd) $($MyInvocation.MyCommand.Name).log"

try {

    Write-Warning "Delete log files older than $LogCycle days"
    Get-ChildItem -Path $PSScriptRoot | Where-Object {($Now - $_.LastWriteTime).Days -gt $LogCycle -and $_.extension -eq ".log"} | Remove-Item

    Write-Host "Crawl for log entries"
    $Events = Get-EventLog -LogName $LogName -After $StartTime -Before $EndTime
    
    $Message = "$($Events.length) events have been found in the $LogName log that are forwarded to the server $ServerName"
    Write-Host $Message
    Write-Log -Path $LogFilePath -Message $Message -Component $MyInvocation.MyCommand.Name -Type Info

    $Events | %{

        switch($_.EntryType) {
            "SuccessAudit" { [Syslog_Severity]$Severity = "Informational" }
        }

        Send-SyslogMessage -Server $ServerName -Message $_.Message -Severity $Severity -Facility ([Syslog_Facility]::logaudit) -Hostname $env:COMPUTERNAME -ApplicationName "EventLog" -Timestamp $_.TimeGenerated -MessageID $_.EventID
    }

} catch {

    Write-Error ($_ | Out-String)
    Write-Log -Path $LogFilePath -Message ($_ | Out-String) -Component $MyInvocation.MyCommand.Name -Type Error
}
```

This examples crawls the security log messages from the last hour and forwards each to the server. Yet only one severity type is supported, the switch statement can be extended easily with additional types. Success and failure of this script are logged locally. Below is screenshot of the log file using CMTrace.

[![Untitled](/wp-content/uploads/2017/10/CMTrace-syslog-entry.png)](/wp-content/uploads/2017/10/CMTrace-syslog-entry.png)

The script is intended to run on an hourly schedule. In case the windows scheduler is used to execute the script make sure that the job runs with the highest privileges available. Otherwise the script won't be able to read the security log.