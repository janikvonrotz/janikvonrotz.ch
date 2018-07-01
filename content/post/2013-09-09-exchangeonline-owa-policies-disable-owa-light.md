---
id: 482
title: 'ExchangeOnline OWA Policies - Disable OWA light'
date: 2013-09-09T09:11:32+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=482
permalink: /2013/09/09/exchangeonline-owa-policies-disable-owa-light/
dsq_thread_id:
  - "1738866391"
image: /wp-content/uploads/2013/08/microsoft-office-365-e1394705447131.jpg
categories:
  - Exchange
  - Office 365
  - PowerShell
tags:
  - disable
  - exchange
  - light
  - office365
  - online
  - owa
  - policies
  - powershell
  - snippet
  - version
---
To alter the Exchange owa policies you can access them Using the Office365 administration site and navigate to the Exchange section. In the default policy editor are only limited options available.

<p style="text-align: center;"><img class="size-full wp-image-486 aligncenter" alt="owa policy settings" src="https://janikvonrotz.ch/wp-content/uploads/2013/09/2013-09-09-09_45_09-Outlook-Web-App-Postfachrichtlinie.png" width="610" height="624" /></p>

<!--more-->

In case you would like f.e. disable the outlook wep app light (which is not available in the web editor), you have to use the almighty PowerShell console.

<ol>
    <li><a href="https://technet.microsoft.com/en-us/library/jj984289(v=exchg.150).aspx" target="_blank">Connect to you ExchangeOnline server</a></li>
    <li>Execute this snippet:</li>
</ol>

[code lang="ps"]

Get-OwaMailboxPolicy -Identity "OwaMailboxPolicy-Default" | Set-OwaMailboxPolicy -OWALightEnabled $false

[/code]

&nbsp;