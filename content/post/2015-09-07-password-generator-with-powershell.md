---
id: 3517
title: Password Generator with PowerShell
date: 2015-09-07T15:03:41+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=3517
permalink: /2015/09/07/password-generator-with-powershell/
dsq_thread_id:
  - "4106282181"
image: /wp-content/uploads/2015/06/PowerShell-logo-e1433137513315.png
categories:
  - PowerShell
tags:
  - authenticate
  - easy
  - generator
  - identity
  - password
  - powershell
  - random
  - remember
  - secure
---
Whenever I had to think of a secure password I followed these steps:

* The right order of vocals and consonants makes it more easy to remember a password.
* And so do three digits of a number.
* Add one uppercase Letter. Likely as the first character.
* Add a dot or another sign to expand to vocabulary even more.
<!--more-->
As I'am using a PowerShell a lot I've created the function/script below to create such a password.

```powershell
function Get-RandomPasswort{
    
    $numbers = 1..9
    $consonants = "b","c","d","f","g","h","k","l","m","n","p","r","s","t","v","w","x","z"
    $nopeletters = "j","q","y"
    $vocals = "a","e","i","o","u"
    $dotsandstuff = ",",".","-"
    $nopedotsandstuff = ";",":","_"

    return (Get-Random $consonants).ToString().ToUpper() + 
    (Get-Random $vocals) + 
    (Get-Random $consonants) + 
    (Get-Random $numbers) +  
    (Get-Random $numbers) + 
    (Get-Random $numbers) + 
    (Get-Random $vocals) + 
    (Get-Random $consonants) + 
    (Get-Random $vocals) + 
    (Get-Random $dotsandstuff)
}
```

Get the lastest version of this snippet here: [https://gist.github.com/f437d7764a43b9de6c3a](https://gist.github.com/f437d7764a43b9de6c3a)

### Example

Create 10 new password is easily done.

```
1..10 | %{Get-RandomPasswort}
Zov683oxa.
Pev541imi-
Pim276ovo-
Vus211ixi.
Dab998inu.
Dup268awi.
Det893ema.
Xok762axa-
Lun996azu,
Gal196ere.
```