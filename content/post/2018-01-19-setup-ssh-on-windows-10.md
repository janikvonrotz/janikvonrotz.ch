---
title: Setup SSH on Windows 10
date: 2018-01-19T11:55:48+00:00
author: Janik von Rotz
slug: setup-ssh-on-windows-10
images:
  - /wp-content/uploads/2018/01/windows-10-ssh-client-help.png
categories:
  - System tooling
tags:
  - git
  - putty
  - ssh
  - windows
---
I have started a new job and as usual had to set up a new computer. Using Windows in a Unix environment seems like a bad choice at first. However, Microsoft has changed strategy and embraced the Unix world with projects such as Windows Subsystem for Linux (WSL) or Windows SQL Server support for Linux. In result you can install bash with [WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10) and use the native ssh client. But there is also another way. If you install [Git for Windows](https://git-scm.com/download/win) a ssh binary is shipped as well. This binary and other tools such as `ssh-keygen` are not available from the command line by default. I want show you how to fix this, setup a keypair and start using ssh in a Windows environment.
<!--more-->

First install [Git for Windows](https://git-scm.com/download/win).

Then add the folder `C:/Program Files/Git/usr/bin` to the `PATH` systen environment variable.

Open a PowerShell session.

Try out `ssh`. It should be available now.

Create a new ssh keypair by entering `ssh-keygen -t rsa -b 4096 -C _COMMENT_`

The keys are stored under `/c/Users/_USERNAME_/.ssh/id_rsa`. 

Make sure to create a backup of your private key and encrypt your drive.