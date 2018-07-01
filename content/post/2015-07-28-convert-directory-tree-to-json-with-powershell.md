---
id: 3421
title: Convert Directory Tree to Json with PowerShell
date: 2015-07-28T15:07:23+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=3421
permalink: /2015/07/28/convert-directory-tree-to-json-with-powershell/
dsq_thread_id:
  - "3979634859"
image: /wp-content/uploads/2015/06/PowerShell-logo-e1433137513315.png
categories:
  - PowerShell
tags:
  - children
  - convert
  - data
  - directory
  - get-childitem
  - json
  - powershell
  - tree
---
Today I wrote a simple script that converts a directory tree query with `get-childitem` into a json formatted data tree.
<!--more-->
 
```ps
function Add-Tabstops{
    param($Count)
    $tabs = ""
    for($i=0; $i -lt $Count; $i++){$tabs += "  "}
    return $tabs
}

function Output-JsonChildren{
    param($Path, $Level = 1)
    return $(Get-ChildItem -Path $Path | Where-Object{$_} | ForEach-Object{
        (Add-Tabstops $Level) +
        "{`n" + 
        (Add-Tabstops ($Level+1)) +
        "`"name`"`: `"$($_.Name)`"," + 
        "`n" +
        (Add-Tabstops ($Level+1)) + 
        "`"children`": ["+ 
        $(if($_.psiscontainer){"`n" + (Output-JsonChildren -Path $_.FullName -Level ($Level+2))+ "`n" + (Add-Tabstops ($Level+1))}) +
        "]`n" + 
        (Add-Tabstops ($Level)) +
        "}"
    }) -join ",`n"
}

$JSON = Output-JsonChildren -Path "C:\Documents"

"["
$JSON
"]"
```

Find the latest version of this script here: [https://gist.github.com/7c9e9ea0acd77c965b79](https://gist.github.com/7c9e9ea0acd77c965b79)

The output looks something like this:

```
[
  {
    "name": "Administration",
    "children": [
      {
        "name": "Organisation",
        "children": [
          {
            "name": "Available Abrevations.md",
            "children": []
          },
          {
            "name": "Kürzel Gemeinde und Organisationen.md",
            "children": []
          }
        ]
      },
```