---
id: 4429
title: XKCD PowerShell password generator
date: 2017-08-11T13:04:45+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4429
permalink: /2017/08/11/xkcd-powershell-password-generator/
image: /wp-content/uploads/2015/06/PowerShell-logo-e1433137513315.png
categories:
  - PowerShell
tags:
  - generator
  - list
  - memorable
  - nouns
  - password
  - powershell
  - random
  - remember
  - secure
  - udpate
  - word
---
> Through 20 ears of effort, we've successfully trained everyone to use passwords that are hard for humans to remember, but easy for computers to guess.

If you are a [Hacker News](https://news.ycombinator.com/) binge reader such as me, you might have read this article: [The Guy Who Invented Those Annoying Password Rules Now Regrets Wasting Your Time](http://gizmodo.com/the-guy-who-invented-those-annoying-password-rules-now-1797643987). 

This article reminded my of how stupid current password guidelines are and that we need to change that. A few months ago I wrote a random [password generator function](https://janikvonrotz.ch/2015/09/07/password-generator-with-powershell/) and decided that it needs an update to make the generated passwords more memorable.
<!--more-->
Inspiration for the new random password was the following XKCD comic:

![Untitled](https://janikvonrotz.ch/wp-content/uploads/2017/08/password_strength.png)

So instead of using any kind of character for the password, simply put four random words together.
To create this kind of password we need a word list with common nouns. I've created a german and english word list for you to download:

[wordlist.de.txt](https://janikvonrotz.ch/wp-content/uploads/2017/08/wordlist.de_.txt)
[wordlist.en.txt](https://janikvonrotz.ch/wp-content/uploads/2017/08/wordlist.en_.txt)

**Get-XKCDPassword.ps1**

And here is the PowerShell function:

```powershell
function Get-XKCDPassword {

    Param(
        [int]$words = 4,

        [string]$delimiter = "-",

        [ValidateSet("en","de")] 
        [string]$lang = "en",

        [switch]$FirstLetterUpperCase  
    )
    
    $password = ""
    $wordlist = @{
        de = "https://janikvonrotz.ch/wp-content/uploads/2017/08/wordlist.de_.txt"
        en = "https://janikvonrotz.ch/wp-content/uploads/2017/08/wordlist.en_.txt"
    }
    
    switch($words) {
        {$_ -ge 6 } { throw "Word parameter cannot be greater or equal 6." }
        5 { $range = (3,4) }
        4 { $range = (4,5) }
        3 { $range = (5,6) }
        2 { $range = (7,8) }
        {$_ -le 1 } { throw "Word parameter cannot be less or equal 1." }
    }

    $list = (((Invoke-WebRequest $wordlist[$lang]).Content -split "`n" | ForEach-Object{ 
        New-Object PSObject -Property @{
            Value = $_.ToLower()
            Length=$_.length
        }
    }) | Where-Object { ($_.Length -eq ($range[0] + 1)) -or ($_.Length -eq ($range[1] + 1)) })

    1..$words | ForEach-Object {
        $part =  (Get-Random $list).Value.Trim()

        if($FirstLetterUpperCase ) {
             $password += ((Get-Culture).TextInfo).ToTitleCase($part)
        } else {
            $password += $part
        }

        if($_ -lt $words){ 
            $password += $delimiter 
        }
    }

    return $password
}

Get-XKCDPassword -words 4 -delimiter "-" -lang "de" -FirstLetterUpperCase 
```

Here are some results:
```
bite-bomb-pupil-basis
crime-wedge-alley-roll
side-filly-bloom-fairy
read-tribe-mouth-jail
knot-make-child-moth
```

And a few more in german:
```
Seite-Sorge-Saft-Nadel
Fluss-Glas-Alter-Rand
Nord-Maul-Wert-Nabel
Sand-Schal-Anruf-Rand
Sahne-Post-Name-Park
```