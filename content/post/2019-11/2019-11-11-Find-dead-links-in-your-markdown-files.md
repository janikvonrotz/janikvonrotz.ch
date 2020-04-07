---
title: "Find dead links in your markdown files"
slug: find-dead-links-in-your-markdown-files
date: 2019-11-11T22:24:02+01:00
categories:
 - Scripting
tags:
 - powershell
 - markdown
images:
 - /images/markdown.png
---

The markdown ecosystem is growing. Markdown has become an universal standard with various extensions. Software repositories, websites, documentation systems, all store their content in markdown. This blog is written in markdown as well.

Of course there are downsides to markdown. Portability comes at the cost of missing features. Thus you have to built them on your own. If you run a blog you probably reference tons of other stuff on the internet. Links that point to usefult resources or nowhere. I wanted to find out which links in my markdown documents are dead.
<!--more-->

I ended up with this simple Powershell function:

```powershell
function Test-MarkdownLinks([String]$Path){

    $unreachable = @()

    # Get markdown files recursively
    $files = Get-ChildItem -Path $path -Recurse -Include "*.md"

    $files | %{
        $fileName = $_.Name
        Write-Host "Analyzing $fileName"

        $urls = Select-String -Path $_  -Pattern "\[.+\]\((http.*?)\)" | %{$_.matches.Groups[1]} | Select 

        $urls | %{
            $url = $_.Value
            Write-Host "Requesting url $url"

            try {
                $request = Invoke-WebRequest -Uri $url
            } catch {
                Write-Error "Found dead url $url in $fileName"
                $unreachable += $url
            }
        }
    }

    # Output urls
    return $unreachable
}

$deadlinks = Test-MarkdownLinks -Path "/Users/janikvonrotz/awesome-powershell"
```

The function finds all markdown files recursively from a folder, regexes all http(s) urls and then runs a simple get request. If it fails the url is treated as dead. At the end of the process the function returns all dead urls.
