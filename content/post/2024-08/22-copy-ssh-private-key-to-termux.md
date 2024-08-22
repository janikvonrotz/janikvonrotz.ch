---
title: Copy SSH private key to Termux
slug: copy-ssh-private-key-to-termux
date: 2024-08-22T08:40:10+02:00
categories:
  - System Tooling
tags:
  - android
  - ssh
  - crypto
  - 3-tier
images:
  - /images/Termux.png
draft: false
---
[Termux](https://termux.dev/en/) is a beautiful terminal emulator Android app. As I am working mostly on the command line I wanted to be able to do the same on phone. And most importantly in a case of emergency where I don't have access to my computer I want to able to access my servers and run some commands. This requires to setup existing SSH private keys in the Termux.

<!--more-->

I haven't found a good tutorial that explains how to do this. It's actually very easy.

First you need to able to access your private key file from your phone. In my case I uploaded the file `id_ed25519` to my private Nextcloud. On my phone I access the file and choose Termux to open the file.

Termux then shows this dialog:

![](/images/Save%20file%20in%20Termux.png)

You choose *Open Directory*. The file is saved in `~/download`.

Before we move the file to the correct location. Make sure that you have installed the openssh package.

```bash
apt update && apt upgrade
pkg install openssh
```

This will create the ssh folder in your home directory and provide the `ssh` command.

You can then move the file.

![](/images/Move%20file%20in%20Termux.png)

Now you can access your server using the private ssh key.

![](/images/SSH%20to%20server%20in%20Termux.png)

Note that you cannot change the username in Termux and therefore need to provide the username on the server in the ssh command.