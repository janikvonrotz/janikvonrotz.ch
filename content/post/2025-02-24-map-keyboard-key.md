---
title: Map Keyboard Key
slug: map-keyboard-key
date: 2025-02-24T07:36:04+01:00
categories:
  - Desktop
tags:
  - linux
  - terminal
  - keyboard
images:
  - /images/keychron-keyboard.png
draft: false
---
I am using a "Keychron K3 Pro" keyboard. Right of the space key there is a "Super" key to run the launcher (similar to windows key). Therefore the "Alt Right" key is missing and this makes creating umlauts more difficult. I am using the "Enlgish (int, with AltGr dead keys)" keyboard layout and pressing <kbd>Alt Right</kbd>+<kbd>Shift Right</kbd>+<kbd>"</kbd> gives me the <kdb>¨</kbd>. Remapping a key in linux is very easy.

<!--more-->

To identify the keys we need the scancode. You can run the following command to output the scancodes of the pressed keys:

```bash
sudo showkey --scancodes
```

The first code, in my case `0xe0`, is the super key.

I use `hwdb` (hardware database) to remap the key. Create a new config file.

```bash
sudo vi /etc/udev/hwdb.d/70-keyboard.hwdb
```

And enter this input:

```bash
evdev:input:*
 KEYBOARD_KEY_$SCANCODE=rightalt
```

In my case it was:

```
evdev:input:*
 KEYBOARD_KEY_0xe0=rightalt
```

Reload the hardware database and the key is mapped.

```
sudo systemd-hwdb update
sudo udevadm trigger
```