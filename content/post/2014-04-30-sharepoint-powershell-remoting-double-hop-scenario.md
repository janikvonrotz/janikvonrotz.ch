---
id: 2140
title: SharePoint PowerShell remoting double hop scenario
date: 2014-04-30T08:34:14+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=2140
permalink: /2014/04/30/sharepoint-powershell-remoting-double-hop-scenario/
dsq_thread_id:
  - "2650848952"
image: /wp-content/uploads/2013/07/PowerShell.png
categories:
  - PowerShell
  - SharePoint
tags:
  - double
  - hop
  - powershell
  - remoting
  - scenario
  - sharepoint
---
PowerShell remoting can be difficult to understand. There are many things to configure and unknown limitations. With multiple server to access from one remoting session it gets even more difficult.

For example my latest issue with a three tier SharePoint environment. I thought I configured everything correctly:
<!--more-->

* The user of the remote machine is administrator of the SharePoint machines.
* The user of the remote machine is farm administrator.
* The user of the remote machine is db_owner and shell_access on the SharePoint config database.
* Both environments with remote PowerShell enabled.
* The user of the remote machine is allowed to login on the remote machine.

However I still received an error when I've executed a SharePoint cmdlet via remoting session.

```powershell
$Session = New-PSSession -ComputerName <ComputerName> -Credential (Get-Credential)
Invoke-Command -Session $Session -ScriptBlock {param ($Name) Add-PSSnapin -Name $Name} -ArgumentList "Microsoft.SharePoint.PowerShell"
Invoke-Command -Session $Session -ScriptBlock {Get-SPSite}
```

```powershell
Cannot access the local farm. Verify that the local farm is properly configured, currently available, and that you
have the appropriate permissions to access the database before trying again.
    + CategoryInfo          : InvalidData: (Microsoft.Share...SPCmdletGetSite:SPCmdletGetSite) [Get-SPSite], SPCmdletE
   xception
    + FullyQualifiedErrorId : Microsoft.SharePoint.PowerShell.SPCmdletGetSite

```

I've figuered that's about the PowerShell remoting double hop issue. The remote SharePoint server can't authenticate against the SQL server with the credentials I'm providing from the remoting shell.

This is only possible with the CredSSP authentication type. Here are the steps to configure CredSSP for a remote session.

On the server run:
```powershell
Enable-WSManCredSSP -Role Server
```

On the client run:
```powershell
Enable-WSManCredSSP -Role Client -DelegateComputer <server FQDN>
```

Now you should be able to run the SharePoint cmdlets on the client. Don't forget to provide the new authentication type for the `New-PSSession` command.

```powershell
$Session = New-PSSession -ComputerName <ComputerName> -Credential (Get-Credential) -Authentication Credssp
Invoke-Command -Session $Session -ScriptBlock {param ($Name) Add-PSSnapin -Name $Name} -ArgumentList "Microsoft.SharePoint.PowerShell"
Invoke-Command -Session $Session -ScriptBlock {Get-SPSite}
```