---
title: 'Install SharePoint 2013 Three-tier Farm - Deploy the SQL Server Backup Job'
date: 2014-02-07T10:24:03+00:00
author: Janik von Rotz
slug: install-sharepoint-2013-three-tier-farm-deploy-the-sql-server-backup-job
dsq_thread_id:
  - "2232801594"
image: /wp-content/uploads/2014/02/SQL-Server-e1393417625647.png
categories:
  - SQL Server
tags:
  - backup
  - installation
  - maintenance
  - sharepoint
  - sql server
  - three tier
---
*This post of is part of my [Install SharePoint 2013 Three-tier Farm](https://janikvonrotz.ch/projects/install-sharepoint-2013-three-tier-farm/) project.*

A clean SQL Server maintenance plan should include three processes, backup, integrity check and index optimize.

You could simply set up a SQL Server maintenance plan within the SQL Server management studio. But be aware that there are better solutions, E.g. <a href="https://ola.hallengren.com/">Ola Hallengren's SQL Server Maintenance Solution</a> is my absolutely favourite.

<!--more-->

Ola Hallengren is a Microsoft <a href="https://mvp.microsoft.com/en-us/MVP/Ola%20Hallengren-5000459">MVP </a>who has simplified the installation of SQL Server maintenance plans. There are many other MVP's who even recommend his solutions over the SQL Server management solution:

<blockquote>Ola Hallengren publishes a free set of database maintenance scripts that are like maintenance plans that went to college, married into a nice family, and went to training school afterwards.
</blockquote>

[Brent Ozar](https://www.brentozar.com/archive/2012/02/webcast-recording-dba-darwin-awards-index-edition/)

However the installation of the SQL ServerMaintenance Solution is very easy. A little bit more difficult is the scheduling, that's why I create a clone repository for Ola's SQL scripts and added and example of how to schedule these jobs.

You'll find the repository on GitHub: <a href="https://github.com/janikvonrotz/SQL-Server-Maintenance">https://github.com/janikvonrotz/SQL-Server-Maintenance</a>