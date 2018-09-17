---
title: Making Exchange Distribution Lists Externally Available
date: 2013-11-12T08:36:52+00:00
author: Janik von Rotz
slug: making-exchange-distribution-lists-externally-available
dsq_thread_id:
  - "1958615900"
images:
  - /wp-content/uploads/2013/08/exchange-2013-e1393417827333.jpg
categories:
  - Exchange
tags:
  - distribution lists
  - exchange
  - powershell
---
This might not be best practise but based on the situation you as administrator have to enable this feature.

Making distribution lists externally available is great for spammers, so if you enable this feature, do this as less as possible.

To enable a exchange distribution list for external use I recommand to use this simple PowerShell command:

```powershell
Set-DistributionGroup <groupname> -RequireSenderAuthenticationEnabled $false
```