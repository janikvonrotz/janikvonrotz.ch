---
title: Create GPG Keys
date: 2014-04-09T08:30:29+00:00
author: Janik von Rotz
slug: create-gpg-keys
dsq_thread_id:
  - "2597967325"
image: /wp-content/uploads/2014/03/GnuPG-Logo.png
categories:
  - Ubuntu Server
tags:
  - create
  - encryption
  - generate
  - gnupg
  - gpg
  - key
  - private
  - public
  - symmetric
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9543913](https://gist.github.com/9543913).*  

# Introuction

GPG keys are used for symmetric key encryption.
GnuPG is the most common tool to create such keys.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [dGnuPG, rng-tools](https://janikvonrotz.ch/2014/03/25/install-ubuntu-packages/)

# Instructions

Change the shell context to the user which uses the new GPG keys.

    su [user]
    
Or use the root user.

    sudo su

Give your server some work, otherwhise gpg won't be able to generator random bytes.

    sudo rngd -r /dev/urandom
    
Genrate the gpg key.

    gpg --gen-key
    
Answert the prompts.
    
    Your selection?: (1) RSA and RSA (default)
    What keysize do you want?: 2048
    Key is valid for?: 0 = key does not expire
    Is this correct?: y
    Real name: [firstname] [surname]
    Email address: [mail]@[example.org]
    Comment:
    Change ... (O)kay/(Q)uit?: O
    Enter passphrase: [gpg passphrase]
    Repeat passphrase: [gpg passphrase]
  
Kill the rngd task.

    sudo service rng-tools stop

Show the new GnuPG keys.

    gpg -k

The `gpg key id` is displayed in the line `pub   2048R/>>C58886FB<< 2014-03-14`

Export the public key into a text file and back it up in a secure place.

    gpg --armor --export -a [gpg key id] > [firstname][surname][server name]#public.key

Export the private key into a text file and back it up in a secure place.

    gpg --armor --export-secret-keys -a [gpg key id] > [firstname][surname][server name]#private.key

Exit the user shell context if you have switched to another user.

    exit

Store the `gpg passphrase` in a secure place f.g. [KeePass Password Safe](http://keepass.info/).

# Source

[Unattended, Encrypted, Incremental Network Backups by Kellen](http://www.debian-administration.org/articles/209#d0e109)
[Ubuntu: How to create a lot of entropy for GPG key generation from command line](http://blog.mypapit.net/2011/11/ubuntu-cli-create-entropy-gpg-key.html)