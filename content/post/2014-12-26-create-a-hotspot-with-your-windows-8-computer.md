---
title: Create a hotspot with your windows 8 computer
date: 2014-12-26T14:33:02+00:00
author: Janik von Rotz
slug: create-a-hotspot-with-your-windows-8-computer
images:
  - /wp-content/uploads/2014/12/windwows-8-wlan.png
categories:
  - Windows
tags:
  - hotspot
  - network
  - wlan
---
This short guide will tell you how you can turn your computer into a hotspot to share a internet connection with you wlan devices.

We assumed that you either have your computer connected with the internet by a cable or a mobile data connection, or you have not internet connection at all and just want create a LAN (local computer network).
<!--more-->
First of all your wifi adapter has to able to share a connection, you can check that by opening the PowerShell commandline (Windows + R and enter "powershell") then enter the following command to get a summary of your wlan driver configuratons.

	netsh wlan show drivers

Then look for this property:

	Hosted network supported  : Yes

It should say yes, otherwhise it's not possible for your wlan adapter to share a network.

Now let's create the hosted network by entering this command:

	netsh wlan set hostednetwork mode=allow ssid=<network name> key=<passkey>

Here's an example:

	netsh wlan set hostednetwork mode=allow ssid=adhocjanik key=12345678

Make sure you have administrator privilegs when doing so (if not, search the powershell programm, make a right click on it and select "run as administrator").

If everything went fine, you can start your hosted network with this command:


	netsh wlan start hostednetwork
