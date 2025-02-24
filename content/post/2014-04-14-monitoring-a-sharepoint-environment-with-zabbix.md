---
title: Monitoring a SharePoint Environment with Zabbix
date: 2014-04-14T15:16:08+00:00
author: Janik von Rotz
slug: monitoring-a-sharepoint-environment-with-zabbix
images:
  - /wp-content/uploads/2014/04/Zabbix-Logo-e1397484832591.png
categories:
  - Web server
tags:
  - monitoring
  - reporting
  - sharepoint
  - zabbix
---
*This post of is part of my [Install SharePoint 2013 Three-tier Farm](https://janikvonrotz.ch/projects/install-sharepoint-2013-three-tier-farm/) project.*

Zabbix is an OpenSource monitoring tool.
And offers everything from simple storage availabilty reporting up to end to end web service calls.
It's extensible with plugins and shell scripts. There isn't a thing you can't monitor with Zabbix.
<!--more-->
In our company we use Zabbix to monitor the SharePoint infrastructure.
One of the great features for this case is the web monitor. Zabbix can request specific websites and authenticate against the webserver.
That's why I don't need an IIS warm up script for our SharePoint infrastructure. Zabbix always makes a request against the SharePoint website after an IIS recycle.

In case you already have a running Zabbix server you can monitor your SharePoint web application with this web rule:

![SharePoint 2013 - Zabbix web monitoring scenario](/wp-content/uploads/2014/04/SharePoint-2013-Zabbix-web-monitoring-scenario.jpg)

![SharePoint 2013 - Zabbix web monitoring step](/wp-content/uploads/2014/04/SharePoint-2013-Zabbix-web-monitoring-step.jpg)
