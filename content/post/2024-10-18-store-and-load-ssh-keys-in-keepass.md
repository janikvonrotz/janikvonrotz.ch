---
title: Store and load SSH keys in KeePass
slug: store-and-load-ssh-keys-in-keepass
date: 2024-10-18T11:03:41+02:00
categories:
  - Security
tags:
  - keepass
  - ssh
  - secure
  - key
images:
  - /images/KeePassXC.png
draft: false
---
I learned about this KeePass feature way too late. With KeePass you can store and load your SSH keys in a secure and encrypted way. No more worrying about your SSH private key being exposed or accessed on your local machine.

<!--more-->

KeePass can communicate with the SSH agent. It is a feature that needs to be enabled:

* Open KeePass and navigate to *Tools > Settings*
* Select *SSH Agent* on the sidebar and click *Enable SSH Agent integration*

You should get a notice *SSH Agent connection is working!*.

Lets store the SSH private key in KeePass:

* Open your KeePass database and create a new entry
* Define a custom title such as `SSH Key $YOURNAME`
* If the SSH private key is encrypted, store its password
* Then open the *Advanced* section and upload your SSH private key as attachment:

![](/images/KeePass-Attachments-SSH-Key.png)

* Now open the *SSH Agent* section and under *Private key* select *attachment*
* Select your key file

Now everything is ready to load the SSH private key with the agent. 

* Ensure that the `.ssh` folder does not contain any key
* Open KeePass and right click the SSH key entry
* Click *Add key to SSH agent*
* Open the command line and test the SSH connection with `ssh -T git@github.com`

You can customize the behaviour of the SSH Agent plugin in many ways. For example you can automatically load s specific SSH key if the database unlocked. 
