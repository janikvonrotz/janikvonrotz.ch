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
 
[code lang="powershell"]
function Add-Tabstops{
    param($Count)
    $tabs = &quot;&quot;
    for($i=0; $i -lt $Count; $i++){$tabs += &quot;  &quot;}
    return $tabs
}

function Output-JsonChildren{
    param($Path, $Level = 1)
    return $(Get-ChildItem -Path $Path | Where-Object{$_} | ForEach-Object{
        (Add-Tabstops $Level) +
        &quot;{`n&quot; + 
        (Add-Tabstops ($Level+1)) +
        &quot;`&quot;name`&quot;`: `&quot;$($_.Name)`&quot;,&quot; + 
        &quot;`n&quot; +
        (Add-Tabstops ($Level+1)) + 
        &quot;`&quot;children`&quot;: [&quot;+ 
        $(if($_.psiscontainer){&quot;`n&quot; + (Output-JsonChildren -Path $_.FullName -Level ($Level+2))+ &quot;`n&quot; + (Add-Tabstops ($Level+1))}) +
        &quot;]`n&quot; + 
        (Add-Tabstops ($Level)) +
        &quot;}&quot;
    }) -join &quot;,`n&quot;
}

$JSON = Output-JsonChildren -Path &quot;C:\Documents&quot;

&quot;[&quot;
$JSON
&quot;]&quot;
[/code]

Find the latest version of this script here: [https://gist.github.com/7c9e9ea0acd77c965b79](https://gist.github.com/7c9e9ea0acd77c965b79)

The output looks something like this:

[code]
[
  {
    &quot;name&quot;: &quot;Administration&quot;,
    &quot;children&quot;: [
      {
        &quot;name&quot;: &quot;Organisation&quot;,
        &quot;children&quot;: [
          {
            &quot;name&quot;: &quot;Available Abrevations.md&quot;,
            &quot;children&quot;: []
          },
          {
            &quot;name&quot;: &quot;Kürzel Gemeinde und Organisationen.md&quot;,
            &quot;children&quot;: []
          }
        ]
      },
[/code]