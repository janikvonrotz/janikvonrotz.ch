---
title: HHVM on Ubuntu 14.4 LTS fix
date: 2014-09-30T13:33:30+00:00
author: Janik Vonrotz
slug: hhvm-on-ubuntu-14-4-lts-fix
images:
  - /wp-content/uploads/2014/04/HHVM-Logo.png
categories:
  - Web server
tags:
  - error
  - hhvm
  - ubuntu
---
If you'll get this nasty error:

	/usr/bin/hhvm: error while loading shared libraries: libgmp.so.10: cannot open shared object file: No such file or directory

Simply run the following command:

	sudo apt-get install libgmp10
