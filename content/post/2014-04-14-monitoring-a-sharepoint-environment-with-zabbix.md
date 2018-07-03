---
id: 1915
title: Monitoring a SharePoint Environment with Zabbix
date: 2014-04-14T15:16:08+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=1915
permalink: /2014/04/14/monitoring-a-sharepoint-environment-with-zabbix/
dsq_thread_id:
  - "2610883638"
image: /wp-content/uploads/2014/04/Zabbix-Logo-e1397484832591.png
categories:
  - Zabbix
tags:
  - end
  - monitor
  - open
  - reporting
  - service
  - sharepoint
  - site
  - software
  - source
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

<img src="https://janikvonrotz.ch/wp-content/uploads/2014/04/SharePoint-2013-Zabbix-web-monitoring-scenario.jpg" alt="SharePoint 2013 - Zabbix web monitoring scenario" width="499" height="372" class="aligncenter size-full wp-image-1957" />

<img src="https://janikvonrotz.ch/wp-content/uploads/2014/04/SharePoint-2013-Zabbix-web-monitoring-step.jpg" alt="SharePoint 2013 - Zabbix web monitoring step" width="919" height="517" class="aligncenter size-full wp-image-1958" />
