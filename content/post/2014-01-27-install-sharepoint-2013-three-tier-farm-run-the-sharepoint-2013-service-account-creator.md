---
title: 'Install SharePoint 2013 Three-tier Farm - Run the SharePoint 2013 Service Account Creator'
date: 2014-01-27T13:24:53+00:00
author: Janik Vonrotz
slug: install-sharepoint-2013-three-tier-farm-run-the-sharepoint-2013-service-account-creator
images:
  - /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - concept
  - powershell
  - sharepoint
  - sql server
  - three tier
---
*This post of is part of my [Install SharePoint 2013 Three-tier Farm](https://janikvonrotz.ch/projects/install-sharepoint-2013-three-tier-farm/) project.*

As many other Windows services SharePoint and SQL Server requires different service accounts in order to run secure and reliable.

There are different concepts and strategies to create service accounts for a SharePoint installation. One of the most reliable solution I've found <a href="https://absolute-sharepoint.com/2013/01/sharepoint-2013-service-accounts-best-practices-explained.html">here</a>.

<!--more-->

In case the link is broken, these documents will do fine:

<ul>
    <li><a href="/wp-content/uploads/2014/01/SharePoint-2013-Service-Accounts-Cheat-Sheet-Low-Security-Option.pdf">SharePoint 2013 Service Accounts Cheat Sheet-Low Security Option</a></li>
    <li><a href="/wp-content/uploads/2014/01/SharePoint-2013-Service-Accounts-Cheat-Sheet-MediumSecurity-Option.pdf">SharePoint 2013 Service Accounts Cheat Sheet-MediumSecurity Option</a></li>
    <li><a href="/wp-content/uploads/2014/01/SharePoint-2013-Service-Accounts-Cheat-Sheet-High-Security-Option.pdf">SharePoint 2013 Service Accounts Cheat Sheet-High Security Option</a></li>
</ul>

Each level of security contains a range of service accounts. Depending on you content and business requirements, you chose a suitable security level.

You can create those Service Accounts automatically with PowerShell using the <a href="https://sp2013serviceaccount.codeplex.com/" target="_blank">SharePoint 2013 Service Account Creator</a> project on CodePlex.

For this project I've chosen the highest security option including optional accounts.

<h1>Source</h1>

<a href="https://technet.microsoft.com/en-us/library/cc263445.aspx" target="_blank">https://technet.microsoft.com/en-us/library/cc263445.aspx</a>