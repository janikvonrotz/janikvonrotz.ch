---
id: 720
title: Open a Windows Remote Connection using KeePass credentials
date: 2013-11-13T15:15:16+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=720
permalink: /2013/11/13/open-a-windows-remote-connection-using-keepass-credentials/
dsq_thread_id:
  - "1962619221"
image: /wp-content/uploads/2013/11/KeePassIcon-e1393417316329.png
categories:
  - KeePass
tags:
  - credentials
  - desktop
  - keepass
  - mstsc
  - remote
---
KeePass is a highly recommended Passwordsafe. Despite its supposed to be used mainly by private people it's adaptable for business cases. In my company the KeePass password database is saved on a SharePoint folder and is encrypted with a password and a private key. The key has to be stored on the local machine.

It could be difficult to force employees to store their passwords in the KeePass database as many won't get along with it. They'll more likely store their password in third party tools. 
However storing a users password in another programm as KeePass f.e. microsoft remote desktop can be a security risk because the password is only encrypted in the user context.
<!--more-->
With a KeePass I found an easy solution to open a remote connection with credentials from a KeePass entry. The url field of an entry can be abused to run windows commands an pass parameters from  the KeePass entry such as password, username or even custom fields.

This example shows how to configure an entry in order to open a remote connection using the stored credentials:

```
cmd://"C:\Windows\System32\cmd.exe" /c cmdkey.exe /generic:TERMSRV/{S:SERVER} /user:{S:DOMAIN}{USERNAME} /pass:{PASSWORD} &amp; mstsc.exe /v:{S:SERVER} &amp; cmdkey.exe /delete:TERMSRV/{S:SERVER}
```

Latest version of this code snippet: [https://gist.github.com/7449352](https://gist.github.com/7449352)

[![KeePass Url RDP Connection 1](https://janikvonrotz.ch/wp-content/uploads/2013/11/KeePass-Url-RDP-Connection-1.png)](https://janikvonrotz.ch/wp-content/uploads/2013/11/KeePass-Url-RDP-Connection-1.png)

[![KeePass Url RDP Connection 2](https://janikvonrotz.ch/wp-content/uploads/2013/11/KeePass-Url-RDP-Connection-2.png)](https://janikvonrotz.ch/wp-content/uploads/2013/11/KeePass-Url-RDP-Connection-2.png)

[![KeePass Url RDP Connection 3](https://janikvonrotz.ch/wp-content/uploads/2013/11/KeePass-Url-RDP-Connection-3.png)](https://janikvonrotz.ch/wp-content/uploads/2013/11/KeePass-Url-RDP-Connection-3.png)