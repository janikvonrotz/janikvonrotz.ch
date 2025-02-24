---
title: "Another Vim Cheat Sheet"
slug: another-vim-cheat-sheet
date: 2019-06-06T11:31:39+02:00
categories:
 - Unix
tags:
 - vim
 - cheat sheet
 - man page
images:
 - /images/vim-logo.png
---

Getting started with Vim is not easy. Fueled by an entirely different ecosystem of plugins, scripts and obscure workflows it is difficult to see the advantage of using Vim. But, to watch experienced people using Vim is always impressive and makes one eager to learn it.

This cheat sheet does not intend to provide you with a list of plugins and dotfiles, but aims at giving an overview of the most common commands. I assmelbed this document while learning to vim.
<!--more-->

In order to unsertand these command I recommend to read about the following Vim topics first:

* leader key
* plugins with plug
* modes and buffers

# Sections

## Plugins

Plugins are managed by vim plug. My vim config including the list of plugins is available at [GitHub - janikvonrotz/dotfiles](https://github.com/janikvonrotz/dotfiles).

### fzf vim

Open file with fzf in new split or tab.

```bash
ctrl + t # tab split
ctrl + x # split
ctrl + v # vsplit
```

Search git files with key bindings.  
`leader + f`

### fugitive

Resolve merge conflicts.

```bash
# open conflicted file

# run diff command
:Gdiff

# navigate between remote and local version

# use diff put to select changes
dp

# write changes
:Gwrite
```

Checkout file from index.  
`:Gread`

Show file history.  
`:Gblame`

Rebase interactive.

```bash
# Rebase last 10 commits
git rebase -i HEAD~10

# Navigate and replace word
cw

# Enter letter for commands
d
```

### vim indent guides

Show indents.  
`leader + ig`

## Basics

Save current file.  
`:w`

Override current file.  
`:w!`

Save and quit.  
`:wq` or `ZZ`

Reload current file.  
`:e`

Quit vim.  
`:q`

Quite all splits and tabs.  
`:qa`

## Profile

Reload vim profile.  
`:so $MYVIMRC` or `:source %`

## Modes

Enter visual character selection mode.  
`v`

Enter visual line selection mode.  
`V`

Enter insert mode.
`i`

Enter insert mode after current character.  
`a`

Exit mode.  
`ctrl + c` or `ESC`

## Navigation

Open new or existing file inside vim.  
`:e /path/to/file`

Open file browser inside vim.  
`:e /path/to/direcotry`

Navigate with keys.

```bash
h # left
j # down
k # up
ö # right
w # word
e # end
b # back
$ # end of line
0 # beginning of line
```

Navigate pages.

```bash
ctrl + u # page up
ctrl + d # page down
G # go to bottom
gg # go to top
```

List previous opened files.  
`:jumps`

Open the last file.  
`:e#`

Navigate search results.  

```
n # next
shift + n # previous
```

## Session

Save the current session.  
`:mksession`

Open vim with session file.  
`vim -S Session.vim`

## Edit

Undo changes.  
`u`

Redo changes.  
`ctrl + r`

Copy and paste.  

```bash
# cut current selection
d
# delete word
dw
# copy current selection
y
# cut the current line
dd
# copy current line
yy
# paste
p
```

Search and replace in file.  
`:%s/old/new/g`

Add line below current line.  
`o`

Tell vim to not autoindent or otherwise alter the format of your pasted text.  
`:set paste` or `:set nopaste`

Move line up or down.  
`ddp` and `dd{j,k}p` or `:m{+,-}{i}` and `:m{i}`

## Select

Select text between character.  
`vi{c}`

Select current word.  
`vw`

## Window

Split window vertically.  
`:vs`

Split window horizontally.  
`:sp`

Switch between splits.  
`ctrl + w & {h,j,k,l}`

## Leader

Set leader key.  
`:let mapleader = "-"`

## Tabs

Open file in new tab.  
`:tabnew /path/to/file`

Open multiple files in tabs.

```bash
:args path/to/files/*
:tab all
```

Navigate tabs.  

```bash
gt # go to next tab
gT # go to previous tab
{i}gt # go to tab in position i
```

## Shell

Run shell command in split pane.  
`:new | 0read ! command`

## Buffer

Reload all buffers.  
`:bufdo e`

# Sources

[Learn Vimscript the Hard Way](http://learnvimscriptthehardway.stevelosh.com/chapters/06.html#leader)

[Jesse Leite - It's dangerous to Vim alone! Take Fzf](https://jesseleite.com/posts/2/its-dangerous-to-vim-alone-take-fzf)

[Thoughtbot - Vim Splits - Move Faster and More Naturally](https://thoughtbot.com/blog/vim-splits-move-faster-and-more-naturally)

