---
title: Git branch typing completion and other useful stuff
date: 2018-06-22T09:11:28+00:00
author: Janik von Rotz
slug: git-branch-typing-completion-and-other-useful-stuff
images:
  - /wp-content/uploads/2018/06/git-completion-folder.png
categories:
  - Software development
---
Recently I was looking a way to have auto completion with git when checking out a branch. The solution was kind of obvious. The [github git mirror repository](https://github.com/git/git) has a [folder containing experimental tools](https://github.com/git/git/tree/master/contrib). One of these tools is a shell scripts that enables completion for various git sub commands. Exactly what I was looking for.
<!--more-->

If you also want to make use of this feature, you can download the script with the command below:

```
curl https://raw.githubusercontent.com/git/git/master/contrib/completion/git-completion.bash -o ~/.git-completion.bash
```

Scripts for other shells are available.

If you are bash user like I am, you can enable the completion script in your bash profile with the following command:

```
test -f ~/.git-completion.bash && . $_
```

