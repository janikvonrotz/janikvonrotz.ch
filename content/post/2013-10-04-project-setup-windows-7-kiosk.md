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

[![Windows 7 Kiosk - Setup](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-7-Kiosk-Setup.png)](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-7-Kiosk-Setup.png)

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

[![Windows - Desktopbackground](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Desktopbackground.png)](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Desktopbackground.png)

[![Windows - Taskbar Result](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Taskbar-Result.png)](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Taskbar-Result.png)

<strong>Remove Desktop Icons</strong>

Remove the default desktop icons.

[![Windows - Remove Desktop Icons](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Remove-Desktop-Icons.png)](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Remove-Desktop-Icons.png)

<strong>Remove System Buttons</strong>

Remove the start system buttons and the options showed after click Ctrl + Alt + Delete.

[![Windows - Remove System Buttons](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Remove-System-Buttons.png)](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Remove-System-Buttons.png)

<strong>Restricted Start Menu</strong>

Remove  the all items in the windows start menu.

[![Windows - Restricted Start Menu](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Restricted-Start-Menu.png)](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Restricted-Start-Menu.png)

[![Windows - Start Menu Result](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Start-Menu-Result.png)](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Start-Menu-Result.png)

<strong>Restricted Taskbar</strong>

Denie any possiblity to customize the windows taskbar.

[![Windows - Restricted Taskbar](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Restricted-Taskbar.png)](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Restricted-Taskbar.png)

<strong>SharePoint Icon</strong>

Adds a simple icon to the desktop.

[![Windows - SharePoint Icon](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-SharePoint-Icon.png)](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-SharePoint-Icon.png)

[![Windows - Icon Result](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Icon-Result.png)](https://janikvonrotz.ch/wp-content/uploads/2013/10/Windows-Icon-Result.png)

<h2>Internet Explorer</h2>

<strong>Delete Cache</strong>

Delete the browser cache on exit.

[![Internet Explorer - Delete Cache](https://janikvonrotz.ch/wp-content/uploads/2013/10/Internet-Explorer-Delete-Cache.png)](https://janikvonrotz.ch/wp-content/uploads/2013/10/Internet-Explorer-Delete-Cache.png)

<strong>Disable Save Passwords</strong>

Internet explorer is not allowed to prompt for saving password information.

[![Internet Explorer - Disable Save Passwords](https://janikvonrotz.ch/wp-content/uploads/2013/10/Internet-Explorer-Disable-Save-Passwords.png)](https://janikvonrotz.ch/wp-content/uploads/2013/10/Internet-Explorer-Disable-Save-Passwords.png)

<strong>Hide Menus</strong>

Hide internet explorer menus.

[![Internet Explorer - Hide Menus](https://janikvonrotz.ch/wp-content/uploads/2013/10/Internet-Explorer-Hide-Menus.png)](https://janikvonrotz.ch/wp-content/uploads/2013/10/Internet-Explorer-Hide-Menus.png)

[![Internet Explorer - Result](https://janikvonrotz.ch/wp-content/uploads/2013/10/Internet-Explorer-Result.png)](https://janikvonrotz.ch/wp-content/uploads/2013/10/Internet-Explorer-Result.png)