---
title: Monitor and audit Active Directory user and group management
date: 2017-10-12T15:54:08+00:00
author: Janik von Rotz
slug: monitor-and-audit-active-directory-user-and-group-management
image: /wp-content/uploads/2017/10/Active-Directory-Logo.jpg
categories:
  - Active Directory
tags:
  - audit
  - compliance
  - event
  - group policy
  - log
  - monitor
  - query
  - security
  - traceability
---
Traceability is key when collaborating in the Active Directory (AD). Multiple admins changing and updating permissions and policies makes it difficult being compliant with the company's policies. It is important to monitor mutations in the directory. By default audit policies are disabled for Domain Controllers (DC) and must be enabled explicitly. Enabling auditing for the DCs is quite easy, querying the logs for a specific event is a bit more difficult.

In this guide you'll learn how to enable auditing for a specific case and how to query the audit logs for a specific event.
<!--more-->

The  tutorial assumes that there is a:

* Domain Controller.
* Group policies, security groups, users, ...
* Admins with DC access.

# Enable Auditing

Let's start by have a look on the already enabled audit categories.

* Log into the DC.
* Open PowerShell as admin.
* Run `auditpol /get /category:*`

The command returns a list of audit categories and its status. These settings have been enabled by either the auditpol tool or via GPOs.

In our scenario we would like to track management of users and groups, which is part of the **Audit Account Management**. To enable this audit category create a new group policiy for the DC.

* Open the GPO management console.
* Right-click the *Domain Controllers* organizational unit.
* Create new GPO and open it in the GPO editor.
* Enable logging for subcategories: `Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options > Audit: Force audit policy subcategory settings...`
* Increase the security log size to 4GB: `Computer Configuration > Windows Settings > Security Settings > Event Log > Maximum security log size: 4268032`
* Then navigate to `Computer Configuration > Windows Settings > Security Settings > Advanced Audit Policy Configuration > Account Management`
* Enable the required audit categories.
* Make a `gpupdate /force` on the DC.
* Run `auditpol /get /Category:*` and double-check whether the settings are correct.

If you open the security event log on the DC there should be events logging account management mutations.

Source: [Microsoft Docs - Monitoring Active Directory for Signs of Compromise](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/monitoring-active-directory-for-signs-of-compromise)

# Query Audit Logs

As mentioned querying the event log is a bit more difficult. The event log viewer offers limited features for filtering events and searching by specific keywords. In contrast with PowerShell it is possible to filter and search the event log by any property and keyword.

Here is a simple example:

```powershell
$LogName = "security"
$StartTime = Get-Date("2017-10-12 12:50")
$EndTime = Get-Date("2017-10-12 13:00")
$SearchKey = "username"

Get-WinEvent -FilterHashtable @{LogName=$LogName; StartTime=$StartTime;EndTime=$EndTime} | Where-Object {$_.Message -match $SearchKey} | select Id, TimeCreated, Message | Format-List
```

Source: [Hey, Scripting Guy! Blog - Filtering Event Log Events with PowerShell](https://blogs.technet.microsoft.com/heyscriptingguy/2015/10/20/filtering-event-log-events-with-powershell/)
