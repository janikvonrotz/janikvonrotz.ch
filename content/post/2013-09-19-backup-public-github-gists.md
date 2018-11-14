---
title: Backup Public GitHub Gists
date: 2013-09-19T12:52:32+00:00
author: Janik von Rotz
slug: backup-public-github-gists
images:
  - /wp-content/uploads/2013/09/github.jpg
categories:
  - scripting
tags:
  - download
  - gist
  - git
  - github
  - powershell
  - scripting
---
To manage my code snippets I'm using <a href="https://gist.github.com/janikvonrotz">GitHubGist </a>connected with <a href="https://app.gistboxapp.com">Gistbox</a>.

Sadly none of this services providing a backup nor a download function for the gist files. That's why I came upwith the idea to download them with PowerShell script.

For first my script only can download public gists, because I don't know how to implement an authentication, luckily each of my gists is public. I recommend you to do the same, it's the idea of OpenSource.

So here's the script:

<!--more-->

```powershell
[System.Reflection.Assembly]::LoadWithPartialName("System.Web.Extensions")

function Get-GitHubGists{

    param(
        [string]$UserUrl
    )

    # set up request
    $WebRequest = [System.Net.WebRequest]::Create($UserUrl)
    $WebRequest.Method ="GET"
    $WebRequest.ContentLength = 0

    # get response
    $WebRequest = $WebRequest.GetResponse()
    $WebRequest = new-object System.IO.StreamReader($WebRequest.GetResponseStream())
    $WebResponse = $WebRequest.ReadToEnd()

    # convert json
    $JavaScriptSerializer = New-Object System.Web.Script.Serialization.JavaScriptSerializer
    $GitHubGists = $JavaScriptSerializer.DeserializeObject($WebResponse)

    # download github gists
    $GitHubGists | %{

        if(Test-Path $_.id){

            cd ($_.id)
            Write-Host "Update GitHubGist: $($_.description)"
            git pull
            cd ..

        }else{

            Write-Host "Cloning GitHubGist: $($_.description)"
            git clone ($_.git_pull_url)
        }
    }
}

Get-GitHubGists -UserUrl https://api.github.com/users/janikvonrotz/gists?page=1
Get-GitHubGists -UserUrl https://api.github.com/users/janikvonrotz/gists?page=2
Get-GitHubGists -UserUrl https://api.github.com/users/janikvonrotz/gists?page=3
```

Latest version: <a href="https://gist.github.com/6622205">https://gist.github.com/6622205</a>