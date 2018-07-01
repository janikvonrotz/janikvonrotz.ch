---
id: 698
title: Making Exchange Distribution Lists Externally Available
date: 2013-11-12T08:36:52+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=698
permalink: /2013/11/12/making-exchange-distribution-lists-externally-available/
dsq_thread_id:
  - "1958615900"
image: /wp-content/uploads/2013/08/exchange-2013-e1393417827333.jpg
categories:
  - Exchange
tags:
  - "2010"
  - "2013"
  - available
  - command
  - distribution
  - exchange
  - externally
  - lists
  - making
  - powershell
---
This might not be best practise but based on the situation you as administrator have to enable this feature.

Making distribution lists externally available is great for spammers, so if you enable this feature, do this as less as possible.

To enable a exchange distribution list for external use I recommand to use this simple PowerShell command:

```ps
Set-DistributionGroup <groupname> -RequireSenderAuthenticationEnabled $false
```