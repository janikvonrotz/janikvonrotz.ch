---
id: 1889
title: Installing and Configuring SharePoint 2013 Farm
date: 2014-04-14T09:29:48+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=1889
permalink: /2014/04/14/installing-and-configuring-sharepoint-2013-farm/
dsq_thread_id:
  - "2610183207"
image: /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - best
  - deploy
  - installation
  - practice
  - project
  - server
  - sharepoint
  - three
  - tier
---
*This post of is part of my [Install SharePoint 2013 Three-tier Farm](https://janikvonrotz.ch/projects/install-sharepoint-2013-three-tier-farm/) project.*

This is a shortened version of my best practice for installing a SharePoint server.
<!--more-->
# Local user rights

Group Local Administrators members:
	
* User: Administrator (Built-In)
* Group: Domain\Domain Admins
* User: Domain\SP_Admin
* User: Domain\SP_Farm
	
System configuraton

* Installed PowerShell PowerUp
* Installed English Language Paket
* Disabled UAC
* Disabled Internet Explorer Enhanced Security
* Turn off Windows Firewall for Domain
* Update zone settings

# SQL alias

sp1sql: sp1sql.domain.com

# Install prerequisites

To Install the prerquisisites you should use a PowerShell script from Technet [http://gallery.technet.microsoft.com/DownloadInstall-SharePoint-e6df9eb8](http://gallery.technet.microsoft.com/DownloadInstall-SharePoint-e6df9eb8)

# Installation

Install Microsoft Office Servers on a additional drive.

Don't configure the SharePoint Farm with the built-in wizard.

I recommend to install the  SharePoint Farm with PowerShell only.

The most popular PowerShell install solution for SharePoint is [https://autospinstaller.codeplex.com/](https://autospinstaller.codeplex.com/)

AutoSPInstallation offers a lot features and takes a lot of time to configure it. 
If it's too much I've got another install solution for you [https://gist.github.com/8862266](https://gist.github.com/8862266)

# Update

You can get the latest SharePoint 2013 Updates from this Microsoft site [http://technet.microsoft.com/en-us/sharepoint/jj891062.aspx](http://technet.microsoft.com/en-us/sharepoint/jj891062.aspx)
