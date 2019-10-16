---
title: Using pass in teams
date: 2018-04-03T08:29:22+00:00
author: Janik Vonrotz
slug: using-pass-in-teams
images:
  - /wp-content/uploads/2018/04/multiple-locks.jpg
categories:
  - Security
tags:
  - git
  - gpg
  - password
  - unix
---
# Introduction

[Pass](https://www.passwordstore.org/) is the standard password manager for Unix systems. It follows the [Unix philosophy](http://en.wikipedia.org/wiki/Unix_philosophy).

Pass saves passwords in text files and encrypts them using a gpg key. The folder structure containing the encrypted files is the pass store. Sharing a pass store without handing over the gpg key requires a gpg key exchange. Git is integrated into the pass [cli](https://en.wikipedia.org/wiki/Command-line_interface) and is used as version control system.

This document is a guideline for users which require access to a shared pass store and is also a documentation of how to set up a shared pass store. The first part elaborates the process of creating a shared pass store and the second part shows how collaboration from the perspective of a user looks like.
<!--more-->

## Variables

In the document you will find different variables starting and ending with **_** underscore. Below is a description of what those variables mean.

    $PROJECT_NAME: A unique name for the shared pass store folder.
    $GIT_REMOTE_URL_: Remote url of the shared pass store git repository.
    $PERSONAL_MAIL: Personal mail address which is used as gpg key id.
    $TEAM_MAIL: Shared team mail address which is used for the shared gpg key id.

# Setup a shared pass store

Install pass using the package manager of your choice and setup a new personal store.

Install pass using yum.  
`yum install pass`

If you do not have a personal gpg key, create one. Make sure to set a correct mail address.  
`gpg --gen-key`

Initialize a personal pass store.  
`pass init $PERSONAL_MAIL`

Passwords are encrypted with a gpg key. In a team environment the gpg key should not impersonate somebody but rather the team itself. That is why a new gpg key is required.

```bash
gpg --gen-key
# (1) RSA and RSA (default)
> enter
# What keysize do you want? (2048)
> enter
# 0 = key does not expire
> enter
> y
# Real name:
> $TEAM_MAIL
# Email address:
> $TEAM_MAIL
# Confirm
> O
# Enter the password
> _PASSWORD_
```

Create a new pass store in a subfolder.  
`pass init -p $PROJECT_NAME $TEAM_MAIL`

Add the gpg test password to the new store.  
`pass generate $PROJECT_NAME/sharedpass 20`

Setup a git repo for the shared pass store.

```bash
cd ~/.password-store/$PROJECT_NAME
git init
git add .
git commit -m "Init $PROJECT_NAME password store"
git remote add origin $GIT_REMOTE_URL_
git push --set-upstream origin master
```

Add the gpg public key to the store.

```bash
mkdir .gpg-keys
gpg --output .gpg-keys/$TEAM_MAIL.gpg --export $TEAM_MAIL
git add $TEAM_MAIL.gpg
git commit -m "Add $TEAM_MAIL public key"
git push
```

The remote repo can be cloned as a subfolder into the existing pass store folders. Git will treat the subfolder as a git submodule.

# Request access to shared pass store

In the following steps we assume that another member of the team requires access to the shared pass store.

Clone the shared pass store repository.  
`git clone $GIT_REMOTE_URL_  ~/.password-store/$PROJECT_NAME`

The shared pass store is now available as subfolder to your personal pass store. If you have enabled git for the personal pass store the subfolder will be treated as git submodule. Changes in the shared pass store will not affect your personal store.

In order to be able to decrypt the passwords in the shared pass store, the store key must be signed by the personal gpg key.

List your gpg keys.  
`gpg --list-key`

Add the personal key to the shared pass store.  
`echo "$PERSONAL_MAIL" >> ~/.password-store/$PROJECT_NAME/.gpg-id`

And export the public gpg key.  
`gpg --output ~/.password-store/$PROJECT_NAME/.gpg-keys/$PERSONAL_MAIL.gpg --export $PERSONAL_MAIL`

Then submit a request by committing the gpg id.

```bash
cd ~/.password-store/$PROJECT_NAME
git add .gpg-id
git commit -m "Request access for $PERSONAL_MAIL"
git push
```

The receiver of this request is the owner of the shared pass store signer key `$TEAM_MAIL`.

# Grant access to users

The owner of the signer key must sign all keys of new team members.  

First import the new keys.

```bash
cd ~/.password-store/$PROJECT_NAME/.gpg-keys
gpg --import $PERSONAL_MAIL.gpg
```

Then sign the imported key.

```bash
gpg --edit-key $PERSONAL_MAIL
# sign it
> lsign
> y
# enter the passphrase for $TEAM_MAIL
> save
```

Reinitialize the password store.  
`pass init -e -p $PROJECT_NAME $(cat ~/.password-store/$PROJECT_NAME/.gpg-id)`

Commit the changes.

```bash
cd ~/.password-store/$PROJECT_NAME
git add .
git commit -m "Access granted for $PERSONAL_MAIL"
git push
```

The user should now be able the decrypt the password using the `$PERSONAL_MAIL` key.

# Contribute to a shared pass store

New password insertions must be encrypted with the `$TEAM_MAIL` gpg key.

The gpg key can be imported from the shared pass folder.

```bash
cd ~/.password-store/$PROJECT_NAME/.gpg-keys
gpg --import $TEAM_MAIL.gpg
```

In order to encrypt new pass entries, you must trust the key.

```bash
gpg --edit-key $TEAM_MAIL.gpg
> trust
> 5
# exit the cli
```

Now you can start creating new entries using the [pass cli](https://git.zx2c4.com/password-store/about/).

# Notes

If there is a gpg registry you can obtain the public keys from there. There would be no need to exchange public key as files.

# Updates

*2019-10-16 Edit: Renamed the variables with a '$' prefix. Ensuring bash compatiblity.*

# Sources

[Official pass homepage](https://www.passwordstore.org)

[Medium - Using pass in a team](https://medium.com/@davidpiegza/using-pass-in-a-team-1aa7adf36592)