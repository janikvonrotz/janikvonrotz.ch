---
title: ILIAS Synchronising Tool
date: 2014-10-17T18:27:19+00:00
author: Janik von Rotz
slug: ilias-synchronising-tool"
categories:
  - Scripting
tags:
  - ilias
  - powershell
  - syncing
---
My university [Hochschule Luzern](http://hslu.ch/) uses [ILIAS](http://www.ilias.de/docu/goto.php?target=root_1) as an open source e-learning and content management platform.

The main part of the publishing process for ILIAS looks like this: The lectureres publish their content (presentations, scripts, software, ..) to ILIAS and the students download it afterwards. It is possible to set up a notification for related changes. However the students still have to copy the files to the local computer manually.

To make this a bit easier, I've developed a synchronisation tool with PowerShell. It consists of four script files:
<!--more-->
**HSLUDriveConfig.ps1**
Cofiguration file for WebDAV mount and sync task.

**Mount-Drives.ps1**
Mount the ILIAS directories of your choice to a local drive letter automatically.

**Sync-Drives.ps1**
Start the synchronisation task.
![ILIAS Sync](/wp-content/uploads/2014/10/ILIAS-Sync-1024x674.png)

**Dismount-Drives.ps1**
Dismount the local ILIAS drives.

You can download the latest versions of these files here: [https://gist.github.com/janikvonrotz/67e36431069bcc778262](https://gist.github.com/janikvonrotz/67e36431069bcc778262)

To mount and sync your ILIAS directory follow the steps:

1. Customize the config file. Add your username, update the paths, names and letters and choice which folders you want to sync.

2. open your PowerShell console by looking for "PowerShell" in your windows search.

3. Execute this command on the console: `Set-ExecutionPolicy Unrestricted`

4. Now close PowerShell and double click the Mount script. It should mount your ILIAS drives now.

5. If everything went well, run the synchronisation script now. Every sync task should ask for confirmation. It was very important for me that I know what's new in the directory and wether there's conflict with an existing file or not.

Hopefully everything works fine, no guarantee. Always backup your data before syncing!