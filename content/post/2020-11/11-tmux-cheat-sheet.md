---
title: "Tmux Cheat Sheet"
slug: tmux-cheat-sheet
date: 2020-11-11T11:01:47+01:00
categories:
 - Unix
tags:
 - tmux
 - cheat sheet
 - man page
images:
 - /images/tmux-logo.png
---

Tmux is a terminal multiplexer for Unix-like operating systems. It allows multiple terminal sessions to be accessed simultaneously in a single window. It is useful for running more than one command-line program at the same time. It can also be used to detach processes from their controlling terminals, allowing SSH sessions to remain active without being visible.
<!--more-->

## Install

See https://github.com/tmux/tmux/wiki/Installing for details.

## Basics

Start new session.  
`tmux`

Attach to last session.  
`tmux a`

Show time.  
<kbd>ctrl + b</kbd> <kbd>t</kbd>

Kill session.  
`tmux kill-session`

## Window

Create new window.  
<kbd>ctrl + b</kbd> <kbd>c</kbd>

Rename new window.  
<kbd>ctrl + b</kbd> <kbd>,</kbd>

Close window.  
<kbd>ctrl + b</kbd> <kbd>&</kbd>

Select window.  
<kbd>ctrl + b</kbd> <kbd>[0-9]</kbd>

List windows.  
`ctrl + b w`

Enter window index.  
`ctrl + b '`

## Pane
Split vertically.  
<kbd>ctrl + b</kbd> <kbd>%</kbd>

Toggle last active pane.  
<kbd>ctrl + b</kbd> <kbd>;</kbd>

Split horizontally.  
<kbd>ctrl + b</kbd> <kbd>"</kbd>

Switch to pane to the direction.  
<kbd>ctrl + b</kbd> <kbd>[h,j,k,l]</kbd>

Close current pane.  
<kbd>ctrl + b</kbd> <kbd>x</kbd>

Move pane right.  
<kbd>ctrl + b</kbd> <kbd>}</kbd>

Move pane left.  
<kbd>ctrl + b</kbd> <kbd>{</kbd>

Convert pane into a window.  
<kbd>ctrl + b</kbd> <kbd>!</kbd>

Resize  pane vertically.
<kbd>ctrl + b + ↑</kbd>
<kbd>ctrl + b + ↓</kbd>

Resize  pane horizontally.
<kbd>ctrl + b + ←</kbd>
<kbd>ctrl + b + →</kbd>

## Session
Detach from session.  
<kbd>ctrl + b</kbd> <kbd>d</kbd>

Attach to last session.  
`tmux a`

Rename session.  
<kbd>ctrl + b</kbd> <kbd>$</kbd>

List sessions.  
<kbd>ctrl + b</kbd> <kbd>s</kbd>

Attach to named session.  
`tmux a -t name`

Start new session.  
<kbd>ctrl + b</kbd> <kbd>:new</kbd>

## Copy Mode

Enter copy mode.  
<kbd>ctrl + b</kbd> <kbd>[</kbd> or <kbd>v</kbd>

Search down.  
<kbd>/</kbd>

Search up.  
<kbd>?</kbd>

Next keyword occurance.    
<kbd>n</kbd>

Navigate up and down.  
<kbd>ctrl + u</kbd> <kbd>ctrl + d</kbd>

Start selection.  
`space`

Copy selection.  
`enter`

Paste selection.  
<kbd>ctrl + b</kbd> <kbd>]</kbd>

List buffers.  
<kbd>ctrl + b</kbd> `:list-buffer`

Paste buffer selection.  
`choose-buffer`

Save buffer.  
`save-buffer buf.txt`

Return to normal mode.  
<kbd>ctrl + c</kbd>

## Plugins
### tmux-resurrect

Save session.  
<kbd>ctrl + b</kbd> <kbd>ctrl + s</kbd>

Restore session.  
<kbd>ctrl + b</kbd> <kbd>ctrl + r</kbd>
