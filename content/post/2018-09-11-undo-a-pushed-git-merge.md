---
title: "Undo a pushed git merge"
slug: undo-a-pushed-git-merge
date: 2018-09-11T18:37:25+02:00
categories:
 - Software development
tags:
 - git
images:
  - /wp-content/uploads/2014/03/Git-Logo.png
---

First of all the solution to the problem I am tackling should not exist in the first place. If you have to revert a merge on a git branch, think about alternatives. If not other option is left, I'll show you how you can undo a merge with `git revert`.
<!--more-->

First checkout your branch that has been targeted by the merge.

`git checkout master`

Have a look into the git log and look for the hash of the merge commit.

`git log`

Now undo the merge using the revert command.

`git revert -m 1 _COMMIT_HASH_`

As mentioned undoing is a dangerous process. Depending on how far the merge commit is from the head of your branch you end up with a lot of conflicts.

Check the git repo for conflicts.

`git status`

Once you have resolved the issues you can commit the revert commit.

`git commit`

Congrats! You did something were bad 😠