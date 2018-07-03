---
id: 516
title: Handling user password change and expiration issues with Office365 and ADFS – Part 2
date: 2013-09-23T08:50:49+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=516
permalink: /2013/09/23/handling-user-password-change-and-expiration-issues-withoffice365-and-adfs-part-2/
dsq_thread_id:
  - "1788553262"
image: /wp-content/uploads/2013/08/microsoft-office-365-e1394705447131.jpg
categories:
  - Active Directory
  - Blog
  - Office 365
  - Web development
tags:
  - adfs
  - change
  - expiration
  - handling
  - managing
  - office365
  - part
  - password
  - reset
---
<a href="https://janikvonrotz.ch/2013/08/08/handling-user-password-change-and-expiration-issues-with-office365-and-adfs-part-1/">https://janikvonrotz.ch/2013/08/08/handling-user-password-change-and-expiration-issues-with-office365-and-adfs-part-1/</a>

This is part two of my experience in handling the password change office365 architecture issue.

Last time I've built a simple script  to notificate the users about the status of their passwords. In the mean time we (me and another employ of the "<a href="https://vbl.ch" target="_blank">vbl </a>Informatik") built a simple website for the office365 users to change their password.

<!--more-->

The whole project is now available on <a href="https://github.com/janikvonrotz/ActiveDirectory-Password-Change">https://github.com/janikvonrotz/ActiveDirectory-Password-Change</a>

Checkout the instructions in the readme fileto set up the password change website.

If you have setup the website already I recommend you to include the site with a sharepoint webpart here's an example:

![Passwortwechsel](https://janikvonrotz.ch/wp-content/uploads/2013/09/Passwortwechsel.png)

<h1>Requirements</h1>

<ul>
    <li>A webserver supporting the php ldap module</li>
    <li>bower client
<ul>
    <li>node.js</li>
    <li>npm package manager</li>
</ul>
</li>
    <li>An ActiveDirectory user who has the right to reset a user's password</li>
</ul>