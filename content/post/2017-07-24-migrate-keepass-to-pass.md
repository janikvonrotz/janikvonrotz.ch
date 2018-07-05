---
id: 4399
title: Migrate KeePass to Pass
date: 2017-07-24T10:05:46+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4399
permalink: /2017/07/24/migrate-keepass-to-pass/
image: /wp-content/uploads/2017/07/Pass-Unix-password-manager.png
categories:
  - KeePass
tags:
  - chrome
  - cli
  - commandline
  - keepass
  - mac
  - manager
  - migration
  - pass
  - password
  - solution
  - standard
  - unix
  - windows
---
I'm using [KeePass](http://keepass.info/) for a few years now. It always has been the password manager of my choice. 
Currently I'm using KeePass on my Mac and Windows connected to the same database file. The KeePass database file is stored in a OneDrive folder, encrypted with a password and keyfile, which is stored in the [Keybase](https://keybase.io) filesystem. This setup gives me maximum security and portability. However, it makes it impossible to use KeePass on my mobile device. Also I miss the possibility to use KeePass in my browser or on the command line. I've looked for an alternative solution, which doesn't compromise on security and gives me the same level of portability. 

<!--more-->

So I've found [Pass](https://www.passwordstore.org/). The standard Unix password manager. Pass is a command line tool that stores password entries in gpg-encrypted files in a git folder. There is a Windows, Mac, iOS client and a browser plugin. I've decided to give it a try. Now I would like to show you how I migrated my KeePass database into the Pass store.

1. First I've exported the database as an XML file. This is only possible with the Windows KeePass client.

2. On my Mac I've set up Pass with a store. You could setup Pass on Windows with [Bash on Ubuntu on Windows](https://msdn.microsoft.com/en-us/commandline/wsl/about) aka Linux subsystem for windows.

3. Finally I developed and executed a [PowerShell](https://github.com/PowerShell/PowerShell) script on my Mac, which migrates the xml data to Pass.

**keepass2pass.ps1**

```powershell
[xml]$Content = Get-Content -Path "KeepassData.xml"

function Traverse-Tree ($Node, $ParentPath) {

    $Path = $ParentPath + "/" + $Node.Name
    $ChildNodes = $Node.Group
    $Entries = $Node.Entry

    if($Entries -and ($Node.Name -notcontains "Recycle Bin")) {
        foreach ($Entry in $Entries) {
            $Content = ""
            $Title = ""
            $Password = ""
            foreach ($Field in $Entry.String) {
                switch($Field.Key) {
                    "Title" { $Title = $Field.Value }
                    "Password" { $Password = $Field.Value.'#text' }
                    default {
                        $Content += "$($Field.Key): `"$($Field.Value)`"`n"
                    }
                }
            }
            @{
                Path = $Path + "/" + $Title
                Content = $Password + "`n" + $Content
            }
        }
    }

    if($ChildNodes) {
        foreach ($ChildNode in $ChildNodes ) {
            Traverse-Tree $ChildNode $Path
        }
    }
}

$PasswordEntries = $Content.KeePassFile.Root.Group.Group | ForEach-Object {
    Traverse-Tree -Node $_ -ParentPath ""
} | ForEach-Object {
    Write-Host "Creating pass entry: $($_.Path)"
    $_.Content | &amp; pass insert -m $_.Path
}
```

You can get the latest version of this file here: [https://gist.github.com/898be0891058ee091dae8c68a8e2b91e](https://gist.github.com/898be0891058ee091dae8c68a8e2b91e)

The script is fairly simple. It traverses the database folder tree and extracts all password entries. These entries are then recreated with the `pass insert` command.

Feel free to use and update the script.