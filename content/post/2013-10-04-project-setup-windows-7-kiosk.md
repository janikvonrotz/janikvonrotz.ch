---
id: 548
title: 'Project: Setup Windows 7 Kiosk'
date: 2013-10-04T16:01:56+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=548
permalink: /2013/10/04/project-setup-windows-7-kiosk/
dsq_thread_id:
  - "1824260455"
image: /wp-content/uploads/2013/10/Group-Policy.jpg
categories:
  - Active Directory
tags:
  - access
  - activedirectory
  - desktop
  - disable
  - explorer
  - features
  - group
  - hide
  - internet
  - kiosk
  - policies
  - policy
  - restricted
  - setup
  - windows
---
The goal of this project is a simple Windows 7 Kiosk installation with nothing else as the newest version of internet explorer installed. A user should not be allowed to do something than can malfunction the system or even elevating the user privileges. I want to show you in this post which GroupPolicies I've used and what configurations I made to set up this type of installation.

First I want to commit my principles for working with ActiveDirectory and Group Policies:

<ul>
    <li>If not needed a GroupPolicy shouldn't contain any registry keys.
<ul>
    <li>Group Policies instructions are much easier to read.</li>
</ul>
</li>
    <li>Only AMDX templates are allowed, this means no AMD templates or anything else.
<ul>
    <li>AMDX won't in contrast to AMD templates becopied to the client, they stay in the SYSVOL Policy Definition folder on the domain controller.</li>
</ul>
</li>
    <li>The Group Policy objects should be reusable.</li>
    <li>Configuring the minimum.</li>
</ul>

<!--more-->

<h1>Preview</h1>

The logged in user can...

<ul>
    <li>visit websites</li>
    <li>printing a document</li>
    <li>searching the internet</li>
    <li>and lock the computer, this was not supposed to be enabled, but I couldn't find a way yet to disable this feature.</li>
</ul>

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-7-Kiosk-Setup.png"><img class="aligncenter size-full wp-image-564" alt="Windows 7 Kiosk - Setup" src="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-7-Kiosk-Setup.png" width="1279" height="943" /></a>

<h1>Setup</h1>

The setup of the windows workstation is very simple:

<ul>
    <li>Windows 7 Profession</li>
    <li>Internet Explorer 10</li>
</ul>

The Group Policy Management Console is equipped with newest Windows 7 AMDX templates.

<h1>Group Policies</h1>

The following section shows the policies I've used to restrict the access to the computer and it's programs.

<h2>Windows 7</h2>

<strong>Desktop background</strong>

Add a desktop wallpaper.

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Desktopbackground.png"><img class="aligncenter size-full wp-image-552" alt="Windows - Desktopbackground" src="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Desktopbackground.png" width="740" height="233" /></a>

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Taskbar-Result.png"><img class="aligncenter size-full wp-image-562" alt="Windows - Taskbar Result" src="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Taskbar-Result.png" width="506" height="245" /></a>

<strong>Remove Desktop Icons</strong>

Remove the default desktop icons.

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Remove-Desktop-Icons.png"><img class="aligncenter size-full wp-image-553" alt="Windows - Remove Desktop Icons" src="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Remove-Desktop-Icons.png" width="376" height="183" /></a>

<strong>Remove System Buttons</strong>

Remove the start system buttons and the options showed after click Ctrl + Alt + Delete.

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Remove-System-Buttons.png"><img class="aligncenter size-full wp-image-554" alt="Windows - Remove System Buttons" src="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Remove-System-Buttons.png" width="371" height="566" /></a>

<strong>Restricted Start Menu</strong>

Remove  the all items in the windows start menu.

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Restricted-Start-Menu.png"><img class="aligncenter size-full wp-image-555" alt="Windows - Restricted Start Menu" src="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Restricted-Start-Menu.png" width="365" height="606" /></a>

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Start-Menu-Result.png"><img class="aligncenter size-full wp-image-560" alt="Windows - Start Menu Result" src="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Start-Menu-Result.png" width="410" height="517" /></a>

<strong>Restricted Taskbar</strong>

Denie any possiblity to customize the windows taskbar.

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Restricted-Taskbar.png"><img class="aligncenter size-full wp-image-556" alt="Windows - Restricted Taskbar" src="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Restricted-Taskbar.png" width="360" height="693" /></a>

<strong>SharePoint Icon</strong>

Adds a simple icon to the desktop.

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-SharePoint-Icon.png"><img class="aligncenter size-full wp-image-557" alt="Windows - SharePoint Icon" src="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-SharePoint-Icon.png" width="752" height="331" /></a>

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Icon-Result.png"><img class="aligncenter size-full wp-image-561" alt="Windows - Icon Result" src="https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Icon-Result.png" width="124" height="115" /></a>

<h2>Internet Explorer</h2>

<strong>Delete Cache</strong>

Delete the browser cache on exit.

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/10/Internet-Explorer-Delete-Cache.png"><img class="aligncenter size-full wp-image-549" alt="Internet Explorer - Delete Cache" src="https://janikvonrotz.ch/wp-content/uploads/2013/10/Internet-Explorer-Delete-Cache.png" width="422" height="256" /></a>

<strong>Disable Save Passwords</strong>

Internet explorer is not allowed to prompt for saving password information.

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/10/Internet-Explorer-Disable-Save-Passwords.png"><img class="aligncenter size-full wp-image-550" alt="Internet Explorer - Disable Save Passwords" src="https://janikvonrotz.ch/wp-content/uploads/2013/10/Internet-Explorer-Disable-Save-Passwords.png" width="715" height="285" /></a>

<strong>Hide Menus</strong>

Hide internet explorer menus.

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/10/Internet-Explorer-Hide-Menus.png"><img class="aligncenter size-full wp-image-551" alt="Internet Explorer - Hide Menus" src="https://janikvonrotz.ch/wp-content/uploads/2013/10/Internet-Explorer-Hide-Menus.png" width="417" height="793" /></a>

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/10/Internet-Explorer-Result.png"><img class="aligncenter size-full wp-image-563" alt="Internet Explorer - Result" src="https://janikvonrotz.ch/wp-content/uploads/2013/10/Internet-Explorer-Result.png" width="1089" height="490" /></a>