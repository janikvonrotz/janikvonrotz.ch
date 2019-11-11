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
 - /images/logo.png
draft: true
---


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
```

`Test-MarkdownLinks -Path "/Users/janikvonrotz/awesome-powershell"`