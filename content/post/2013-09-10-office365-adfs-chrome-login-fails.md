---
title: Office365 ADFS Chrome Login fails
date: 2013-09-10T13:49:40+00:00
author: Janik von Rotz
slug: office365-adfs-chrome-login-fails
dsq_thread_id:
  - "1745104946"
images:
  - /wp-content/uploads/2013/08/microsoft-office-365-e1394705447131.jpg
categories:
  - Office 365
tags:
  - adfs
  - authentication
  - browser
  - chrome
  - error
  - login
  - logon
  - multiple
  - office365
  - prompt
---
Today I experienced an exotic behaviour, a client couldn't access his Office365 page due he wasn't able to login on the ADFS authentication prompt.

After googling and binging (just kidding, <em>NERD</em>) I found a simple <a href="https://stackoverflow.com/questions/5436441/adfs-authentication-ie8-works-chrome-fails" target="_blank">solution</a>.

<!--more-->

[caption id="attachment_498" align="aligncenter" width="714"]<a href="/wp-content/uploads/2013/09/2013-09-10-13_24_09-Default-vblw2k12adfs1-Remotedesktopverbindung.png">![adfs disable extended protection](/wp-content/uploads/2013/09/2013-09-10-13_24_09-Default-vblw2k12adfs1-Remotedesktopverbindung.png)</a> To turn Extended Protection off, on the AD FS server, launch IIS Manager, then, on the left side tree view, access Sites -> Default Web Site -> adfs -> ls. Once you’ve selected the "/adfs/ls" folder, double-click the Authentication icon, then right-click Windows Authentication and select Advanced Settings… On the Advanced Settings dialog, choose Off for Extended Protection.[/caption]

&nbsp;

Disabling the extended windows authentication protection solved this issue, but I have to admit I'm not quite sure about services depending on this settings, maybe you'll experience some other errors related to the ADFS service.