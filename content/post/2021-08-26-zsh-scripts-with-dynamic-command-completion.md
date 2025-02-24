---
title: "ZSH scripts with dynamic command completion"
slug: zsh-scripts-with-dynamic-command-completion
date: 2021-08-26T09:43:28+02:00
categories:
 - Blog
tags:
 - zsh
 - bash
 - completion
 - dynamic
images:
 - /images/oh-my-zsh.png
---

ZSH/Bash completion guides sure are confusing. I personally had difficulties wrapping my head around those arbitrary `compdef, _describe, _arguments, compadd`  commands. Moreover, in my case I wanted to complete a script that is placed in multiple folders and should load completion arguments dynamically. Luckily I found a simple solution.

<!--more-->

We assume that zsh and oh-my-zsh are configured.

For every project I create `task` script and a `task.md` file that contains a list of available commands, their options and descriptions.

Running `task help` gives the content of `task.md`. So each task script has various subcommands that perform different tasks.

Here is an example for such a task file.

**/\*\*/task**

```bash
#!/bin/bash

set -eo pipefail

# load env vars
if [ -f .env ]
then
  export $(cat .env | sed 's/#.*//g' | xargs)
fi

function help() {
echo
echo "$1 <command> [options]"
echo
echo "commands:"
echo
column -t -s"|" ./task.md | tail -n +3
echo
}

case "$1" in
    install-docker)
        sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
        | sudo apt-key add - \
        && sudo apt-key fingerprint 0EBFCD88
        sudo add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) \
        stable"
        sudo apt-get update && sudo apt-get install docker-ce docker-ce-cli containerd.io
        sudo usermod -aG docker $USER
        ;;
    install-tools)
        cd ~
        # Install node version manager
        [ ! -d 'n' ] && curl -L https://git.io/n-install | 
        # Install python version manager
        [ ! -d '.pyenv' ] && git clone https://github.com/pyenv/pyenv.git ~/.pyenv
        # Install tmux plugins
        [ ! -d '.tmux' ] && git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
        # Install binaries
        sudo apt install -y zsh vim tmux jq pass htop xclip ripgrep
        ;;
    *)
        help task
        exit 1
esac

```

And here is its help file.

**/\*\*/task.md**

```markdown
| command        | option | description     |
| -------------- | ------ | --------------- |
| install-docker |        | Install docker. |
| install-tools  |        | Install tools.  |

```

The autocompletion should list the available commands on `task <TAB>`. This is done by adding an completion file:

**~/.oh-my-zsh/completions/_task**

```bash
#compdef task

_arguments '1: :->tasks' '*: :_files'
case "$state" in
    tasks)
        args=$(awk -F"|" '{print $2}' ./task.md | tail -n +3 | xargs)
        args="$args help"
        _arguments "1:profiles:($args)"
        ;;
esac

```

Whenever the tabulator is hit after entering `task` the completion script will lookup command in the help file and output them as arguments.

```bash
➜  dotfiles git:(master) task <TAB>
install-docker            install-tools 
```

Easy, isn't it? If the command is completed it will list files.

```bash
➜  dotfiles git:(master) ✗ task install-tools <TAB>
default.vim             oh-my-zsh-completions/ 
```