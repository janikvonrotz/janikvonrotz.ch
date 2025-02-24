---
title: "Create and use nvm rc file"
slug: create-and-use-nvm-rc-file
date: 2022-03-17T16:44:45+01:00
categories:
 - JavaScript
tags:
 - node
 - nvm
 - version management
images:
 - /wp-content/uploads/2014/03/Node.js-Logo.png
---

With [nvm](https://github.com/nvm-sh/nvm) you can easily switch between node versions. When working on multiple project is recommended to create a `.nvmrc` file containing the targeted node version. Here is how.

<!--more-->

First make sure node is on the right version.

```bash
➜  example-project git:(main) node -v
v12.22.6
➜  example-project git:(main) nvm use 14
Now using node v14.18.3 (npm v6.14.15)
➜  example-project git:(main) node -v
v14.18.3
```

Pipe the version into the `.nvmrc` file.

```bash
➜  example-project git:(main) node -v > .nvmrc
```

Then assume you switch to another version.

```bash
➜  example-project git:(main) ✗ nvm use 12
Now using node v12.22.6 (npm v6.14.15)
➜  example-project git:(main) ✗ cat .nvmrc
v14.18.3
```

In the project type `nvm use` and it will automatically load the version in the `.nvmrc` file.

```bash
➜  example-project git:(main) ✗ nvm use
Found '/home/janikvonrotz/jaiunedemande/.nvmrc' with version <v14.18.3>
Now using node v14.18.3 (npm v6.14.15)
```

Repeat this process for all projects.