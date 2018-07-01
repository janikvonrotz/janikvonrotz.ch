---
id: 794
title: 'Install SharePoint 2013 Three-tier Farm &#8211; Architecture and Licensing'
date: 2013-12-10T12:35:07+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=794
permalink: /2013/12/10/install-sharepoint-2013-three-tier-farm-architecture-and-licensing/
dsq_thread_id:
  - "2040751075"
image: /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - "2013"
  - architecture
  - farm
  - license
  - project
  - sharepoint
  - three
  - tier
---
*This post of is part of my [Install SharePoint 2013 Three-tier Farm](https://janikvonrotz.ch/projects/install-sharepoint-2013-three-tier-farm/) project.*

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/12/Network-Scheme-SharePoint-Farm.png"><img class="aligncenter size-full wp-image-1006" alt="Network Scheme SharePoint Farm" src="https://janikvonrotz.ch/wp-content/uploads/2013/12/Network-Scheme-SharePoint-Farm.png" width="987" height="689" /></a>

As the server architecture is already given in this case, I'm only going to show you the hardware requirements of those servers.

<h1>Server additional</h1>

Each server installation provides:

<ul>
    <li>latest service pack</li>
    <li>64 Bit</li>
    <li>resources for less than 1000 users</li>
</ul>

<!--more-->

<h3>FIM 2010 R2 (optional)</h3>

Hardware and Software Requirements: <a href="https://technet.microsoft.com/en-us/library/ff512684(v=WS.10).aspx">https://technet.microsoft.com/en-us/library/ff512684(v=WS.10).aspx</a>
Server: Windows Server 2012 R2
Service: FIM 2010 R2, SQL Server 2010 R2
Name: w2k12fim1
CPU: 64-bit, 4cores
RAM: 8 GB
Disk:  System 80 GB, Data 100 GB (dynamisch)

<h3>FIM Portal (optional)</h3>

Hardware and Software Requirements: <a href="https://technet.microsoft.com/en-us/library/ff512684(v=WS.10).aspx">https://technet.microsoft.com/en-us/library/ff512684(v=WS.10).aspx
</a>Server: Windows Server 2012 R2
Service: FIM 2010 R2 Service Portal
Name: w2k12fim2
CPU: 64-bit, 4 cores
RAM: 4 GB
Disk:  System 80 GB

<h3>SQL 2010 R2</h3>

Hardware and Software Requirements: <a href="https://technet.microsoft.com/en-us/library/cc262485.aspx">https://technet.microsoft.com/en-us/library/cc262485.aspx
</a>Server: Windows Server 2012 R2
Service: SQL Server 2010 R2
DNS (Alias): sp1sql
Name: w2k12spsql1
CPU: 64-bit, 4 cores
RAM: 12 GB
Disk:  System 80 GB, Data 100 GB (dynamic)

<h3>SP 2013 App</h3>

Hardware and Software Requirements: <a href="https://technet.microsoft.com/en-us/library/cc262485.aspx">https://technet.microsoft.com/en-us/library/cc262485.aspx
</a>DNS: sp1app.vbl.ch<a href="https://technet.microsoft.com/en-us/library/cc262485.aspx">
<span style="color: #2b2b2b; line-height: 1.5;">Server: Windows Server 2012 R2
</span></a>Service: SharePoint Server 2013
<span style="line-height: 1.5;">Name: w2k12spapp1
</span>CPU: 64-bit, 4 cores
RAM: 12 GB
Disk:  System 80 GB, Data 100 GB (dynamic)

<h3>SP 2013 Web</h3>

Hardware and Software Requirements: <a href="https://technet.microsoft.com/en-us/library/cc262485.aspx">https://technet.microsoft.com/en-us/library/cc262485.aspx</a>
Server: Windows Server 2012 R2
Service: SharePoint Server 2013
DNS: sharepoint.domain.ch, sp1web.domain.ch
Name: w2k12spweb1
CPU: 64-bit, 4 cores
RAM: 12GB
Disk:  System 80 GB,  Data 100 GB (dynamic)

<h3>OWA</h3>

Hardware and Software Requirements: <a href="https://technet.microsoft.com/en-us/library/jj219435.aspx#software">https://technet.microsoft.com/en-us/library/jj219435.aspx#software</a>
Server: Windows Server 2012 R2
Service: Office Web Apps Server 2013
Name: w2k12was1
DNS: was1.domain.ch
CPU: 64-bit, 4 cores
RAM: 12 GB
Disk:  System 80 GB, Data 50 GB (dynamic)

<h1>Server available</h1>

The installation and deployment of this servers won't be part of this project.

<ul>
    <li>Active Directory</li>
    <li>Active Directory Federation Services</li>
</ul>

<h1> DNS entries</h1>

sp1web.domain.ch <strong>CNAME</strong> w2k12spweb1.domain.ch
sp1app.domain.ch <strong>CNAME</strong> w2k12spapp1.domain.ch
was1.domain.ch <strong>CNAME</strong> w2k12was1.domain.ch

<h1>Licenses</h1>

<ul>
<ul type="disc">
    <li>SharePoint 2013
<ul type="circle">
    <li>1x App</li>
    <li>1x Web</li>
    <li>X user CALS</li>
</ul>
</li>
</ul>
<ul type="disc">
    <li>SQL Server 2010 R2 Datacenter
<ul type="circle">
    <li>1x SharePoint</li>
    <li>1x FIM</li>
</ul>
</li>
</ul>
<ul type="disc">
    <li>Windows Server 2012
<ul type="circle">
    <li>1x FIM Service</li>
</ul>
<ul type="circle">
    <li>1x FIM Portal</li>
    <li>1x SP SQL</li>
</ul>
<ul type="circle">
    <li>1x SP App</li>
    <li>1x SP Web</li>
    <li>1x OWA</li>
</ul>
</li>
</ul>
<ul type="disc">
    <li>FIM 2010
<ul type="circle">
    <li>1x  FIM Service</li>
    <li>1x  FIM Portal</li>
    <li>X user CALS</li>
</ul>
</li>
</ul>
</ul>