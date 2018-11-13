---
title: 'Install SharePoint 2013 Three-tier Farm - Installing and Configuring SharePoint SQL Server'
date: 2014-02-25T13:52:33+00:00
author: Janik von Rotz
slug: install-sharepoint-2013-three-tier-farm-installing-and-configuring-sharepoint-sql-server
images:
  - /wp-content/uploads/2014/02/SQL-Server-e1393417625647.png
categories:
  - Microsoft infrastructure
tags:
  - configuration
  - project
  - sharepoint
  - sql server
  - three tier
---
*This post of is part of my [Install SharePoint 2013 Three-tier Farm](https://janikvonrotz.ch/projects/install-sharepoint-2013-three-tier-farm/) project.*

In this post I'll show my best practice for an new SQL Server to run SharePoint properly.

Let's get started with local user rights.

<!--more-->

<h2>Local User Rights</h2>

The Group Local Administrators members are:<a href="/wp-content/uploads/2014/02/Installing-and-Configuring-SharePoint-SQL-Server.jpg">
</a>

<ul type="disc">
    <li>User: Administrator (Built-In)</li>
    <li>Group: DomainDomain Admins</li>
    <li>User: DomainSP1SQL_Admin</li>
</ul>

Based on you standards and options I have the following system configurations for you:

<h2>System configuration (optional)</h2>

<ul type="disc">
    <li>Installed <a href="https://github.com/janikvonrotz/PowerShell-PowerUp">PowerShell PowerUp</a></li>
    <li>Installed English Language Paket</li>
    <li>Disabled UAC</li>
    <li>Disabled Internet Explorer Enhanced Security</li>
    <li>Turn off Windows Firewall for Domain</li>
</ul>

And now it's time to install the SQL server.

<h2>Installation</h2>

The SQL Server 2008 R2 as in my case requires the .Net Framework 3.5. To install it on Windows server 2012 run this command:

```text
dism /online /enable-feature /featurename:NetFX3 /all /Source:d:sourcessxs /LimitAccess
```

Only after executing this command you're able to install the .Net framework as usual with the wizard.

[![.Net 3.5 Installation WinSev 2012](/wp-content/uploads/2014/02/Net-3.5-Installation-WinSev-2012.jpg)](/wp-content/uploads/2014/02/Net-3.5-Installation-WinSev-2012.jpg)

At this point where the SQL server installation I don't want to provide you a step by step guide, as you might install another version or this guide gets obsolete.

I think the sql server configuration file is much more informative.

```text
;SQLSERVER2008 Configuration File
[SQLSERVER2008]

; Specify the Instance ID for the SQL Server features you have specified. SQL Server directory structure, registry structure, and service names will reflect the instance ID of the SQL Server instance.

INSTANCEID="MSSQLSERVER"

; Specifies a Setup work flow, like INSTALL, UNINSTALL, or UPGRADE. This is a required parameter.

ACTION="Install"

; Specifies features to install, uninstall, or upgrade. The list of top-level features include SQL, AS, RS, IS, and Tools. The SQL feature will install the database engine, replication, and full-text. The Tools feature will install Management Tools, Books online, Business Intelligence Development Studio, and other shared components.

FEATURES=SQLENGINE,FULLTEXT,IS,BC,SSMS,ADV_SSMS

; Displays the command line parameters usage

HELP="False"

; Specifies that the detailed Setup log should be piped to the console.

INDICATEPROGRESS="False"

; Setup will not display any user interface.

QUIET="False"

; Setup will display progress only without any user interaction.

QUIETSIMPLE="False"

; Specifies that Setup should install into WOW64. This command line argument is not supported on an IA64 or a 32-bit system.

X86="False"

; Detailed help for command line argument ENU has not been defined yet.

ENU="True"

; Parameter that controls the user interface behavior. Valid values are Normal for the full UI, and AutoAdvance for a simplied UI.

UIMODE="Normal"

; Specify if errors can be reported to Microsoft to improve future SQL Server releases. Specify 1 or True to enable and 0 or False to disable this feature.

ERRORREPORTING="False"

; Specify the root installation directory for native shared components.

INSTALLSHAREDDIR="C:Program FilesMicrosoft SQL Server"

; Specify the root installation directory for the WOW64 shared components.

INSTALLSHAREDWOWDIR="C:Program Files (x86)Microsoft SQL Server"

; Specify the installation directory.

INSTANCEDIR="C:Program FilesMicrosoft SQL Server"

; Specify that SQL Server feature usage data can be collected and sent to Microsoft. Specify 1 or True to enable and 0 or False to disable this feature.

SQMREPORTING="False"

; Specify a default or named instance. MSSQLSERVER is the default instance for non-Express editions and SQLExpress for Express editions. This parameter is required when installing the SQL Server Database Engine (SQL), Analysis Services (AS), or Reporting Services (RS).

INSTANCENAME="MSSQLSERVER"

; Agent account name

AGTSVCACCOUNT="VBLSP1SQL_Agent"

; Auto-start service after installation.

AGTSVCSTARTUPTYPE="Automatic"

; Startup type for Integration Services.

ISSVCSTARTUPTYPE="Automatic"

; Account for Integration Services: DomainUser or system account.

ISSVCACCOUNT="NT AUTHORITYNetworkService"

; Controls the service startup type setting after the service has been created.

ASSVCSTARTUPTYPE="Automatic"

; The collation to be used by Analysis Services.

ASCOLLATION="Latin1_General_CI_AS"

; The location for the Analysis Services data files.

ASDATADIR="Data"

; The location for the Analysis Services log files.

ASLOGDIR="Log"

; The location for the Analysis Services backup files.

ASBACKUPDIR="Backup"

; The location for the Analysis Services temporary files.

ASTEMPDIR="Temp"

; The location for the Analysis Services configuration files.

ASCONFIGDIR="Config"

; Specifies whether or not the MSOLAP provider is allowed to run in process.

ASPROVIDERMSOLAP="1"

; A port number used to connect to the SharePoint Central Administration web application.

FARMADMINPORT="0"

; Startup type for the SQL Server service.

SQLSVCSTARTUPTYPE="Automatic"

; Level to enable FILESTREAM feature at (0, 1, 2 or 3).

FILESTREAMLEVEL="0"

; Set to "1" to enable RANU for SQL Server Express.

ENABLERANU="False"

; Specifies a Windows collation or an SQL collation to use for the Database Engine.

SQLCOLLATION="Latin1_General_CI_AS_KS_WS"

; Account for SQL Server service: DomainUser or system account.

SQLSVCACCOUNT="VBLSP1SQL_Engine"

; Windows account(s) to provision as SQL Server system administrators.

SQLSYSADMINACCOUNTS="VORDEFINIERTAdministratoren" "VBLSP1SQL_Admin"

; Default directory for the Database Engine backup files.

SQLBACKUPDIR="E:Microsoft SQL ServerBackup"

; Default directory for the Database Engine user databases.

SQLUSERDBDIR="E:Microsoft SQL ServerData"

; Provision current user as a Database Engine system administrator for SQL Server 2008 R2 Express.

ADDCURRENTUSERASSQLADMIN="False"

; Specify 0 to disable or 1 to enable the TCP/IP protocol.

TCPENABLED="1"

; Specify 0 to disable or 1 to enable the Named Pipes protocol.

NPENABLED="0"

; Startup type for Browser Service.

BROWSERSVCSTARTUPTYPE="Disabled"

; Specifies how the startup mode of the report server NT service.  When
; Manual - Service startup is manual mode (default).
; Automatic - Service startup is automatic mode.
; Disabled - Service is disabled

RSSVCSTARTUPTYPE="Automatic"

; Specifies which mode report server is installed in.
; Default value: “FilesOnly”

RSINSTALLMODE="FilesOnlyMode"

; Add description of input argument FTSVCACCOUNT

FTSVCACCOUNT="NT AUTHORITYLOCAL SERVICE"

```

The most important configurations are the data directories and the user service accounts. In the last post we created several service accounts, you can use them now as showed here:

<a style="font-family: Montserrat, sans-serif; font-size: 16px; font-style: normal; color: #16a085;" href="/wp-content/uploads/2014/02/Installing-and-Configuring-SharePoint-SQL-Server.jpg">![Installing and Configuring SharePoint SQL Server](/wp-content/uploads/2014/02/Installing-and-Configuring-SharePoint-SQL-Server.jpg)</a>

You might have seen, the temdb is stored on the C drive and I know as s a best practice you should not have tempdb on the C drive because the OS has the paging file on C by default and you don't want the mixed I/O affecting tempdb performance. But let's be honest this action won't affect our SharePoint installation in any way.

Now you should have installed the SQL server successfully. Open the management studio, log in and execute these statements:

```sql
-- Set Memory

EXEC sys.sp_configure N'show advanced options', N'1'  RECONFIGURE WITH OVERRIDE
GO
EXEC sys.sp_configure N'max server memory (MB)', N'10240'
GO
RECONFIGURE WITH OVERRIDE
GO
EXEC sys.sp_configure N'show advanced options', N'0'  RECONFIGURE WITH OVERRIDE
GO

-- Set-Permission

USE [master]
GO
CREATE LOGIN [VBLsp1_admin] FROM WINDOWS WITH DEFAULT_DATABASE=[master]
GO
EXEC master..sp_addsrvrolemember @loginame = N'domainsp1_admin', @rolename = N'dbcreator'
GO
EXEC master..sp_addsrvrolemember @loginame = N'domainsp1_admin', @rolename = N'securityadmin'
GO

-- Backup Compression

EXEC sys.sp_configure N'backup compression default', N'1'
GO
RECONFIGURE WITH OVERRIDE
GO

-- Max Parallelism

sp_configure 'show advanced options', 1;
GO
RECONFIGURE WITH OVERRIDE;
GO
sp_configure 'max degree of parallelism', 1;
GO
RECONFIGURE WITH OVERRIDE;
GO
```

Last but not least update the SQL server installation.

<h1>Update</h1>

On the site <a href="https://sqlserverbuilds.blogspot.ch/">https://sqlserverbuilds.blogspot.ch/</a> you'll find the latest update definitions for your SQL server installation.

In case you don't know you SQL server version run this command:

```sql
SELECT SERVERPROPERTY('productversion'), SERVERPROPERTY ('productlevel'), SERVERPROPERTY ('edition')
```