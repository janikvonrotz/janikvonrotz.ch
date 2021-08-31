---
title: "Sync Obsidian mobile app with Working Copy git repo"
slug: sync-obsidian-mobile-app-with-working-copy-git-repo
date: 2021-08-31T09:55:35+02:00
categories:
 - Knowledge
tags:
 - obsidian
 - git
 - synchronisation
images:
 - /images/obsidian-banner.png
---

I'm using the [Obsidian](https://obsidian.md/) note taking app for almost anything. Having everything stored in plain text Markdown files makes it easy to sync the Obsidian vault with any file sync provider. However, getting the Obsidian vault on the mobile phone without their syncing service is not so easy. In this guide I will explain how you can sync the Obsidian vault to your phone using git and [Working Copy](https://workingcopyapp.com/).

<!--more-->

This guide assumes that you have the following setup:

* Obisidan vault commited with git
* Remote repository on GitHub, GitLab or somewhere else
* iPhone and Working Copy app with access to remote repository

Lets start 

* Open the Obsidian mobile app and create a new vault with the same name as your desktop vault

![](/images/sync-obsidian-with-git-1.png)

* Go to settings, open the advanced options and set for *Override config folder* the value `.obsidian-mobile`

![](/images/sync-obsidian-with-git-2.png)

* Open Working Copy on your phone, open the repo and click on the top right icon.

![](/images/sync-obsidian-with-git-3.png)

* Select *Setup Folder Sync* and choose the Obsidian vault folder.

![](/images/sync-obsidian-with-git-4.png)

* Now Working Copy will sync the repo with the Obsidian vault folder.

![](/images/sync-obsidian-with-git-5.png)

* Reopen the Obsidian vault on your phone and check if the app indexes the new files.

![](/images/sync-obsidian-with-git-6.png)

* Now you can update files with the Obsidian mobile app and commit the changes with Working Copy.

![](/images/sync-obsidian-with-git-7.png)

The Obsidian mobile and destkop app will store their config separately.