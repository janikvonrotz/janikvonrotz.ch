---
title: Install .NET Framework 3.5 on Windows Server 2012 and Windows Server 2012 R2
date: 2014-03-11T07:30:44+00:00
author: Janik von Rotz
slug: install-net-framework-3-5-on-windows-server-2012-and-windows-server-2012-r2
images:
  - /wp-content/uploads/2014/03/Windows-Server-2012-Logo-e1394519276892.jpg
categories:
  - Microsoft infrastructure
tags:
  - .net
  - server
  - windows
---
To install the .Net Framework 3.5 on Windows Server 2012 and Windows Server 2012 R2 you have to enable the install feature from the online source.

```dism /online /enable-feature /featurename:NetFX3 /all /Source:d:\sources\sxs /LimitAccess
```

Now you should be able to install the feature with server management console.

[![install .Net 3.5 on Windows Server 2012 R2](/wp-content/uploads/2014/03/install-net-3.5-e1394519502205.jpg)](/wp-content/uploads/2014/03/install-net-3.5-e1394519502205.jpg)