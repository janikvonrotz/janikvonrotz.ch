---
title: 'bin/bash^M: bad interpreter'
date: 2013-08-14T08:00:19+00:00
author: Janik von Rotz
permalink: /2013/08/14/binbashm-bad-interpreter/
dsq_thread_id:
  - "1605090780"
image: /wp-content/uploads/2013/08/Linux-Logo.jpg
categories:
  - Ubuntu Server
tags:
  - bash
  - character
  - error
  - linux
  - script
  - unix
---
If you're using windows and linux/unix and your also a system administrator who likes to script. The chances are high that you'll get this error when executing a script on a linux/unix machine that has been made on a windows machine: `bin/bash^M: bad interpreter: No such file or directoy`

The `^M` character is a windows line break, which linux/unix can't interpret. The solution is easy, use `dos2unix [filename]` and everything should work fine.