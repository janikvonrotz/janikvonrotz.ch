---
id: 579
title: SharePoint File Reporting Done Right
date: 2013-10-10T16:33:01+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=579
permalink: /2013/10/10/sharepoint-file-reporting-done-right/
dsq_thread_id:
  - "1843312893"
image: /wp-content/uploads/2013/07/PowerShell.png
categories:
  - PowerShell
  - SharePoint
tags:
  - file
  - incorrect
  - information
  - powershell
  - profile
  - report
  - script
  - sharepoint
  - storage
  - unused
---
To know what type of information a SharePoint Installation is holding could get more difficult as the platform grows.

To set quota templates on the site collection is a good way to keep an eye on the SharePoint storage, but it doesn't' show the real used storage.

In case versioning for files is enabled and/or the users don't check in their files properly these files using space without let you know.

<!--more-->

To get a nice report of unchecked, versioned and "normal" files I've coded a PowerShell script.

Most of the functions are part of my Project: <a title="PowerShell Profile" href="https://github.com/janikvonrotz/Powershell-Profile" target="_blank">PowerShell Profile</a>, so this will be a requirement to run this script.

```ps
$SPWebs = Get-SPWebs
$SPWebs | %{

    $SPWeb = $_

    $SPSite = $_.site.url

    Get-SPLists -Url $_.Url -OnlyDocumentLibraries | %{

        $SPList = $_

        $SPListUrl = (Get-SPUrl $SPList).url

        Write-Progress -Activity &amp;quot;Crawl list on website&amp;quot; -status &amp;quot;$($SPWeb.Title): $($SPList.Title)&amp;quot; -percentComplete ([Int32](([Array]::IndexOf($SPWebs, $SPWeb)/($SPWebs.count))*100))

        Get-SPListItems $_.ParentWeb.Url -FilterListName $_.title | %{

            $ItemUrl = (Get-SPUrl $_).Url

            # files
            New-Object PSObject -Property @{
                ParentWebsite = $SPWeb.ParentWeb.title
                ParentWebsiteUrl = $SPWeb.ParentWeb.Url
                Website = $SPWeb.title
                WebsiteUrl = $SPWeb.Url
                List = $SPList.title
                ListUrl = $SPListUrl
                FileExtension = [System.IO.Path]::GetExtension($_.Url)
                IsCheckedOut = $false
                IsASubversion = $false
                Item = $_.Name
                ItemUrl = $ItemUrl
                Folder = $ItemUrl -replace &amp;quot;[^/]+$&amp;quot;,&amp;quot;&amp;quot;
                FileSize = $_.file.Length / 1000000
            }

            $SPItem = $_

            # file subversions
            $_.file.versions | %{

                $ItemUrl = (Get-SPUrl $SPItem).Url

                New-Object PSObject -Property @{
                    ParentWebsite = $SPWeb.ParentWeb.title
                    ParentWebsiteUrl = $SPWeb.ParentWeb.Url
                    Website = $SPWeb.title
                    WebsiteUrl = $SPWeb.Url
                    List = $SPList.title
                    ListUrl = $SPListUrl
                    FileExtension = [System.IO.Path]::GetExtension($_.Url)
                    IsCheckedOut = $false
                    IsASubversion = $true
                    Item = $SPItem.Name
                    ItemUrl = $ItemUrl
                    Folder = $ItemUrl -replace &amp;quot;[^/]+$&amp;quot;,&amp;quot;&amp;quot;
                    FileSize = $_.Size / 1000000
                }
            }
        }

        # checked out files
        Get-SPListItems $_.ParentWeb.Url -FilterListName $_.title -OnlyCheckedOutFiles | %{

            $ItemUrl = $SPSite + &amp;quot;/&amp;quot; + $_.Url

            New-Object PSObject -Property @{
                ParentWebsite = $SPWeb.ParentWeb.title
                ParentWebsiteUrl = $SPWeb.ParentWeb.Url
                Website = $SPWeb.title
                WebsiteUrl = $SPWeb.Url
                List = $SPList.title
                ListUrl = $SPListUrl
                FileExtension = [System.IO.Path]::GetExtension($_.Url)
                IsCheckedOut = $true
                IsASubversion = $false
                Item = $_.LeafName
                ItemUrl = $ItemUrl
                Folder = $ItemUrl -replace &amp;quot;[^/]+$&amp;quot;,&amp;quot;&amp;quot;
                FileSize = $_.Length / 1000000
            }

        }
    }
} | Export-Csv &amp;quot;Report SharePoint Files.csv&amp;quot; -Delimiter &amp;quot;;&amp;quot; -Encoding &amp;quot;UTF8&amp;quot; -NoTypeInformation
```

Most recent version of this snippet is avialable here: <a href="https://gist.github.com/janikvonrotz/6885934" target="_blank">https://gist.github.com/janikvonrotz/6885934</a>

<strong>Update</strong>: This script is now part of my PowerShell Profile project as a function: <a href="https://github.com/janikvonrotz/Powershell-Profile/blob/master/functions/SharePoint/List/Get-SPListFiles.ps1">https://github.com/janikvonrotz/Powershell-Profile/blob/master/functions/SharePoint/List/Get-SPListFiles.ps1</a>