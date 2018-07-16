---
title: Install SharePoint 2013 Three-tier Farm – Installing and Configuring Office Web Apps Server
date: 2014-03-25T09:18:05+00:00
author: Janik von Rotz
slug: install-sharepoint-2013-three-tier-farm-installing-and-configuring-office-web-apps-server
dsq_thread_id:
  - "2504593554"
image: /wp-content/uploads/2014/03/Office-Web-Apps-Logo.png
categories:
  - Windows Server
tags:
  - apps
  - installation
  - project
  - server
  - sharepoint
  - three tier
---
*This post of is part of my [Install SharePoint 2013 Three-tier Farm](https://janikvonrotz.ch/projects/install-sharepoint-2013-three-tier-farm/) project.*

With Windows Server 2012 the deployment of Offic Web Apps Server got a lot easier.
<!--more-->
# Installation

First install the prerequisites.

	Add-WindowsFeature Web-Server,Web-Mgmt-Tools,Web-Mgmt-Console,Web-WebServer,Web-Common-Http,Web-Default-Doc,Web-Static-Content,Web-Performance,Web-Stat-Compression,Web-Dyn-Compression,Web-Security,Web-Filtering,Web-Windows-Auth,Web-App-Dev,Web-Net-Ext45,Web-Asp-Net45,Web-ISAPI-Ext,Web-ISAPI-Filter,Web-Includes,InkandHandwritingServices,NET-Framework-Features,NET-Framework-Core

Download the Office Web Apps Server Installer from [http://go.microsoft.com/fwlink/p/?LinkId=256561](http://go.microsoft.com/fwlink/p/?LinkId=256561)

And install the Office Web Apps Server

Deploy a new Web Apps Server Farm with PowerShell.

	New-OfficeWebAppsFarm -InternalUrl "http://example.org" -ExternalUrl "http://example.org" -EditingEnabled -AllowHttp:$true

# Update

In order to update a Office Web Apps Server installation you have to remove the existing Office Web Apps Farm.

    Remove-OfficeWebAppsMachine

Now you can install the required updates.

If the installation has been finished create the Office Web Apps Farm again according to your settings.

    New-OfficeWebAppsFarm -InternalUrl "http://example.org" -ExternalUrl "http://example.org" -EditingEnabled -AllowHttp:$true
    
The check wether Office Web Apps are running open your browser on:

    http://example.org/hosting/discovery

# Source

[Microsoft Technet: Deploy Office Web Apps Server](http://technet.microsoft.com/en-us/library/jj219455(v=office.15).aspx)
[Update Office Web Apps Server 2013](https://gist.github.com/janikvonrotz/9364202)