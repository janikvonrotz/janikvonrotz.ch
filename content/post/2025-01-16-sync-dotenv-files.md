---
title: Sync .env files
slug: sync-dotenv-files
date: 2025-01-16T09:04:17+01:00
categories:
  - Software development
tags:
  - dotenv
  - syncing
  - encrypted
images:
  - /images/dotenv-sync.png
draft: false
---
The `.env` file is a common standard to define environment variables and secrets for a software project. When working on multiple machines and in teams, ensuring that the `.env` files are up-to-date is important.

I was looking for a solution to solve this problem. If you duck for "Sync .env files" you will most likely end up on [https://www.dotenv.org/docs/quickstart/sync](https://www.dotenv.org/docs/quickstart/sync). The Dotenv project provides a service for syncing `.env` files. However, their service requires an account and this was out of question in my case.

> How can I sync secrets with my team using git only?

<!--more-->

The solution I found was [pass](https://www.passwordstore.org/). I already talked about this tool and most importantly documented [a way to use pass in teams](https://janikvonrotz.ch/2018/04/03/using-pass-in-teams/). For a software project that uses the [taskfile standard](https://taskfile.build/) you can simply add two new commands: `pass-store-dotenv` and `pass-restore-dotend`

Here are the help entries:

```bash
printf "$COLUMN" "store-dotenv" "" "Store content of .env in pass entry."
printf "$COLUMN" "restore-dotenv" "" "Restore content of .env from pass entry."
```

And the functions:

```bash
PASS_ENTRY=/dotenv/project

function store-dotenv() {
    if [ -f .env ]; then
        echo "Store .env file in pass: $PASS_ENTRY"
        cat .env | pass insert -m -f "$PASS_ENTRY"
    else
        echo "No .env file found."
    fi
}

function restore-dotenv() {
    if pass find "$PASS_ENTRY" >/dev/null; then
        echo "Restore .env file from pass: $PASS_ENTRY"
        pass show "$PASS_ENTRY" > .env
    else
        echo "Pass entry not found."
    fi
}
```

To store the .env file in pass run `task pass-store-dotenv` and `pass git push`. To restore it run `pass git pull` and `task pass-restore-dotenv`. The content of the .env file is stored as a pass entry in the `$PASS_ENTRY` path.