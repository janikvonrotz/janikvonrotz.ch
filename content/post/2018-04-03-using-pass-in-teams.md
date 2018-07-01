---
id: 4840
title: Using pass in teams
date: 2018-04-03T08:29:22+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4840
permalink: /2018/04/03/using-pass-in-teams/
specific_page_layout:
  - default-sidebar
image: /wp-content/uploads/2018/04/multiple-locks.jpg
categories:
  - IT Security
tags:
  - exchange
  - git
  - gpg
  - key
  - manager
  - pass
  - password
  - shared
  - store
  - team
  - unix
---
# Introduction

[Pass](https://www.passwordstore.org/) is the standard password manager for Unix systems. It follows the [Unix philosophy](http://en.wikipedia.org/wiki/Unix_philosophy).

Pass saves passwords in text files and encrypts them using a gpg key. The folder structure containing the encrypted files is the pass store. Sharing a pass store without handing over the gpg key requires a gpg key exchange. Git is integrated into the pass [cli](https://en.wikipedia.org/wiki/Command-line_interface) and is used as version control system.

This document is a guideline for users which require access to a shared pass store and is also a documentation of how to set up a shared pass store. The first part elaborates the process of creating a shared pass store and the second part shows how collaboration from the perspective of a user looks like.
<!--more-->

## Variables

In the document you will find different variables starting and ending with **_** underscore. Below is a description of what those variables mean.

    _PROJECT_NAME_: A unique name for the shared pass store folder.
    _GIT_REMOTE_URL_: Remote url of the shared pass store git repository.
    _PERSONAL_MAIL_: Personal mail address which is used as gpg key id.
    _TEAM_MAIL_: Shared team mail address which is used for the shared gpg key id.

# Setup a shared pass store

Install pass using the package manager of your choice and setup a new personal store.

Install pass using yum.  
`yum install pass`

If you do not have a personal gpg key, create one. Make sure to set a correct mail address.  
`gpg --gen-key`

Initialize a personal pass store.  
`pass init _PERSONAL_MAIL_`

Passwords are encrypted with a gpg key. In a team environment the gpg key should not impersonate somebody but rather the team itself. That is why a new gpg key is required.

[code lang="bash"]
gpg --gen-key
# (1) RSA and RSA (default)
> enter
# What keysize do you want? (2048)
> enter
# 0 = key does not expire
> enter
> y
# Real name:
> _TEAM_MAIL_
# Email address:
> _TEAM_MAIL_
# Confirm
> O
# Enter the password
> _PASSWORD_
[/code]

Create a new pass store in a subfolder.  
`pass init -p _PROJECT_NAME_ _TEAM_MAIL_`

Add the gpg test password to the new store.  
`pass generate _PROJECT_NAME_/sharedpass 20`

Setup a git repo for the shared pass store.

[code lang="bash"]
cd ~/.password-store/_PROJECT_NAME_
git init
git add .
git commit -m "Init _PROJECT_NAME_ password store"
git remote add origin _GIT_REMOTE_URL_
git push --set-upstream origin master
[/code]

Add the gpg public key to the store.

[code lang="bash"]
mkdir .gpg-keys
gpg --output .gpg-keys/_TEAM_MAIL_.gpg --export _TEAM_MAIL_
git add _TEAM_MAIL_.gpg
git commit -m "Add _TEAM_MAIL_ public key"
git push
[/code]

The remote repo can be cloned as a subfolder into the existing pass store folders. Git will treat the subfolder as a git submodule.

# Request access to shared pass store

In the following steps we assume that another member of the team requires access to the shared pass store.

Clone the shared pass store repository.  
`git clone _GIT_REMOTE_URL_  ~/.password-store/_PROJECT_NAME_`

The shared pass store is now available as subfolder to your personal pass store. If you have enabled git for the personal pass store the subfolder will be treated as git submodule. Changes in the shared pass store will not affect your personal store.

In order to be able to decrypt the passwords in the shared pass store, the store key must be signed by the personal gpg key.

List your gpg keys.  
`gpg --list-key`

Add the personal key to the shared pass store.  
`echo "_PERSONAL_MAIL_" >> ~/.password-store/_PROJECT_NAME_/.gpg-id`

And export the public gpg key.  
`gpg --output ~/.password-store/_PROJECT_NAME_/.gpg-keys/_PERSONAL_MAIL_.gpg --export _PERSONAL_MAIL_`

Then submit a request by committing the gpg id.

[code lang="bash"]
cd ~/.password-store/_PROJECT_NAME_
git add .gpg-id
git commit -m "Request access for _PERSONAL_MAIL_"
git push
[/code]

The receiver of this request is the owner of the shared pass store signer key `_TEAM_MAIL_`.

# Grant access to users

The owner of the signer key must sign all keys of new team members.  

First import the new keys.

[code lang="bash"]
cd ~/.password-store/_PROJECT_NAME_/.gpg-keys
gpg --import _PERSONAL_MAIL_.gpg
[/code]

Then sign the imported key.

[code lang="bash"]
gpg --edit-key _PERSONAL_MAIL_
# sign it
> lsign
> y
# enter the passphrase for _TEAM_MAIL_
> save
[/code]

Reinitialize the password store.  
`pass init -e -p _PROJECT_NAME_ $(cat ~/.password-store/_PROJECT_NAME_/.gpg-id)`

Commit the changes.

[code lang="bash"]
cd ~/.password-store/_PROJECT_NAME_
git add .
git commit -m "Access granted for _PERSONAL_MAIL_"
git push
[/code]

The user should now be able the decrypt the password using the `_PERSONAL_MAIL_` key.

# Contribute to a shared pass store

New password insertions must be encrypted with the `_TEAM_MAIL_` gpg key.

The gpg key can be imported from the shared pass folder.

[code lang="bash"]
cd ~/.password-store/_PROJECT_NAME_/.gpg-keys
gpg --import _TEAM_MAIL_.gpg
[/code]

In order to encrypt new pass entries, you must trust the key.

[code lang="bash"]
gpg --edit-key _TEAM_MAIL_.gpg
> trust
> 5
# exit the cli
[/code]

Now you can start creating new entries using the [pass cli](https://git.zx2c4.com/password-store/about/).

# Notes

If there is a gpg registry you can obtain the public keys from there. There would be no need to exchange public key as files.

# Sources

[Official pass homepage](https://www.passwordstore.org)

[Medium - Using pass in a team](https://medium.com/@davidpiegza/using-pass-in-a-team-1aa7adf36592)