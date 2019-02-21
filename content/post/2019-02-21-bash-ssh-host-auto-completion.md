---
title: "Bash SSH host auto completion"
slug: bash-ssh-host-auto-completion
date: 2019-02-21T10:59:37+01:00
categories:
 - System tooling
tags:
 - ssh
 - bash
images:
 - /images/horrible openssh logo.gif
---

By default the ssh command does not support auto completion for host names. However, it stores all hosts you have accessed in the `~/.ssh/known_hosts` file. We can take this data and use it for an auto completion function.
<!--more-->

Create the following file in your home folder (yes, I know, another secret file):

**~/.ssh-completion.bash**

```bash
_complete_ssh_hosts ()
{
    COMPREPLY=()
    cur="${COMP_WORDS[COMP_CWORD]}"
    comp_ssh_hosts=`cat ~/.ssh/known_hosts | \
                    cut -f 1 -d ' ' | \
                    sed -e s/,.*//g | \
                    grep -v ^# | \
                    uniq | \
                    grep -v "\[" ;
            cat ~/.ssh/config | \
                    grep "^Host " | \
                    awk '{print $2}'
            `
    COMPREPLY=( $(compgen -W "${comp_ssh_hosts}" -- $cur))
    return 0
}
complete -F _complete_ssh_hosts ssh
```

Then make it executable:

`chmod +x ~/.ssh-completion.bash`

And source the file in your bash profile.

`test -f ~/.ssh-completion.bash && . $_`

Restart bash and you should get auto completion for the ssh command.

Note: This will not work for Ubuntu as it hashes everything in the known_hosts file.