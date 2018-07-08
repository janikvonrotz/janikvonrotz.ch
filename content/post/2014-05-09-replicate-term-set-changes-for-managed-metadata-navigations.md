---
id: 2229
title: Replicate term set changes for managed metadata navigations
date: 2014-05-09T08:24:47+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=2229
permalink: /2014/05/09/replicate-term-set-changes-for-managed-metadata-navigations/
dsq_thread_id:
  - "2671794656"
image: /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - Blog
---
This post is a supplement to <a href="https://janikvonrotz.ch/2014/04/23/limitations-and-workarounds-for-managed-metadata-navigation-for-multiple-site-collections/" title="Limitations and workarounds for managed metadata navigation for multiple site collections">Limitations and workarounds for managed metadata navigation for multiple site collections</a>.

According to my last post I've updated the navigation for all my site collections to use a term set with pinned terms from the global navigation term set.
What I didn't was the fact that changes in the global navigation term set won't be applied to pinned terms in other sets.

To apply this changes I've written a simple PowerShell function that repins the global navigation terms on the allocated navigation term set.
<!--more-->
```powershell
<#
$Metadata = @{
	Title = "Update SharePoint Web Term Navigation"
	Filename = "Update-SPWebTermNavigation.ps1"
	Description = ""
	Tags = "sharepoint, managed, metadata, navigation"
	Project = ""
	Author = "Janik von Rotz"
	AuthorContact = "https://janikvonrotz.ch"
	CreateDate = "2014-05-09"
	LastEditDate = "2014-05-09"
	Url = ""
	Version = "0.0.0"
	License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/ch/ or 
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#>

function Update-SPWebTermNavigation{

<#
.SYNOPSIS
    Update the term based navigation for a speific SharePoint website.

.DESCRIPTION
    Update the term based navigation for a speific SharePoint website.

.PARAMETER Identity
	Url of the website.

.PARAMETER MMSSite
	Managed metadata site.

.PARAMETER TermStoreName
	Name of the term store that holds the navigation term groups.

.PARAMETER TermGroupName
	Name of the term group that holds the navigation term sets.

.PARAMETER TermSetName
	Name of the term set that hold the navigation terms for this website.

.PARAMETER GlobalNavigationTermGroupName
	Name of the term group that holds the global navigation term set. Only required if this is different from the term group name.

.PARAMETER GlobalNavigationTermSetName
	Name of the global term set where the terms should be pinned from.

.EXAMPLE
	PS C:\> Update-SPWebTermNavigation -Identity "http://SiteXY.example.org" -MMSSite "http://sharepoint.example.org" -TermStoreName "Store1" -TermGroupName "Navigation" -TermSetName "SiteXY" -GlobalNavigationTermSetName "GlobalNavigation"

.EXAMPLE
	PS C:\> Update-SPWebTermNavigation -Identity "http://SiteXY.example.org" -MMSSite "http://sharepoint.example.org" -TermStoreName "Store1" -TermGroupName "Navigation" -TermSetName "SiteXY" -GlobalNavigationTermGroupName "AnotherNavigation" -GlobalNavigationTermSetName "GlobalNavigation"

#>

    [CmdletBinding()]
    param(

        [Parameter(Mandatory=$true)]
        [String]$Identity,

        [Parameter(Mandatory=$true)]
        [String]$MMSSite,
        
        [Parameter(Mandatory=$true)]
        [String]$TermStoreName,
        
        [Parameter(Mandatory=$true)]
        [String]$TermGroupName,
        
        [Parameter(Mandatory=$true)]
        [String]$TermSetName,
        
        [Parameter(Mandatory=$false)]
        [String]$GlobalNavigationTermGroupName,

        [Parameter(Mandatory=$true)]
        [String]$GlobalNavigationTermSetName
    )

    #--------------------------------------------------#
    # modules
    #--------------------------------------------------#

    if(-not (Get-PSSnapin "Microsoft.SharePoint.PowerShell" -ErrorAction SilentlyContinue)){Add-PSSnapin "Microsoft.SharePoint.PowerShell"}
      
    #--------------------------------------------------#
    # main
    #--------------------------------------------------#

    Write-Host "Update term navigation settings for: $Identity"

    # get website and site collection
    $SPWeb = Get-SPWeb $Identity
    $SPSite = $SPWeb.Site

    # get the navigation settings
    $WebNavigationSettings = New-Object Microsoft.SharePoint.Publishing.Navigation.WebNavigationSettings($SPWeb)

    # get the navigation term set
    $SPTaxonomySession = Get-SPTaxonomySession -Site $MMSSite
    $TermStore = $SPTaxonomySession.TermStores[$TermStoreName]
    $TermGroup = $TermStore.Groups[$TermGroupName]
    $TermSet = $TermGroup.TermSets[$TermSetName]

    # get the global navigation term set
    if($GlobalNavigationTermGroupName){
        $GlobalNavigationTermGroup = $TermStore.Groups[$GlobalNavigationTermGroupName]
        $GlobalNavigationTermSet = $GlobalNavigationTermGroup.TermSets[$GlobalNavigationTermSetName]
    }else{
        $GlobalNavigationTermSet = $TermGroup.TermSets[$GlobalNavigationTermSetName]
    }

    # remove all the existing terms
    $TermSet.Terms | ForEach-Object{$_.delete()}
    $TermStore.CommitAll()

     # pin the terms from the master term set
     $GlobalNavigationTermSet.Terms | ForEach-Object{$TermSet.ReuseTermWithPinning($_) | Out-Null}

     # copy the sort order
     $TermSet.CustomSortOrder = $GlobalNavigationTermSet.CustomSortOrder 
     $TermStore.CommitAll()

     # update the navigation settings
     $WebNavigationSettings.GlobalNavigation.Source = 2
     $WebNavigationSettings.GlobalNavigation.TermStoreId = $TermStore.Id
     $WebNavigationSettings.GlobalNavigation.TermSetId = $TermSet.Id

     $WebNavigationSettings.AddNewPagesToNavigation = $false
     $WebNavigationSettings.CreateFriendlyUrlsForNewPages = $false
     $WebNavigationSettings.Update()
}
```

This function is part of my [PowerShell PowerUp](http://janikvonrotz.github.io/PowerShell-PowerUp/) framework.

# Credits

* [http://johnlearnt.blogspot.ch/2013/07/sharepoint-2013-cross-site-managed.html](http://johnlearnt.blogspot.ch/2013/07/sharepoint-2013-cross-site-managed.html)