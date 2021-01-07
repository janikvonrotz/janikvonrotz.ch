---
title: "Git pull overwrite local changes"
slug: git-pull-overwrite-local-changes
date: 2021-01-07T10:23:38+01:00
categories:
 - Software development
tags:
 - git
images:
 - /wp-content/uploads/2014/03/Git-Logo.png
---

A new year a new post. Instead of a new years resolution I've got a git wisdom for you. If you ever have to reset your repository/life use this command:
<!--more-->

```bash
git fetch --all
git reset --hard origin/master
git pull origin master
```

It will fetch the latest changes, reset the local repo and then pull the master branch.
