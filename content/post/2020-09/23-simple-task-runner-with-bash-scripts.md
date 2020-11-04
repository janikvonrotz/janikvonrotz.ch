---
title: "Simple task runner with bash/zsh scripts"
slug: simple-task-runner-with-bash-zsh-scripts
date: 2020-09-23T12:03:24+02:00
categories:
 - Scripting
tags:
 - bash
 - task runner
 - zsh
images:
 - "/images/caution runners.png"
---

When you work on multiple projects with different tech (Docker, npm, python, ..), a common interface to build, start, install or clean the state of the project is a powerful tool. There are various task runners for this job, however, every one of them requires you to install at least one dependency and so must everybody else who wants to use the project. What if we can use a task runner that is preinstalled on every computer? What about bash/zsh?
<!--more-->

For a while I used [Task](https://taskfile.dev) as a task runner and created config files in the project root like this one:

**taskfile.yml**

```yml
version: '2'

tasks:
  up:
    cmds:
      - docker-compose up -d
  stop:
    cmds:
      - docker-compose stop
  run-script:
    cmds:
      - ./scripts/another-script
```

It is a nice solution for a common interface to manage projects. But as mentioned in the intro, that is not good enough. It requires a tool, which most developers are not familiar with. The solution for me was bash/zsh. I simply turned every task file into this:

**task**

```zsh
#!/bin/zsh

set -euo pipefail

function start() {
    docker-compose up -d
}

function stop() {
    docker-compose stop
}

case "$1" in
    start)
        start
        ;;
    stop)
        stop
        ;;
    run-script)
        ./scripts/another-script
        ;;
    *)
        echo $"Usage: $0 {start|stop|run-script}"
        exit 1
esac
```

Create an alias `alias task='./task'`, make the script executable `chmod +x task` and run task commands like this `task start`.

Simple isn't it?
