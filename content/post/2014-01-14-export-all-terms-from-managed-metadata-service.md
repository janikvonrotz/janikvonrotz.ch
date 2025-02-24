---
title: Export all Terms from Managed Metadata Service
date: 2014-01-14T16:18:44+00:00
author: Janik von Rotz
slug: export-all-terms-from-managed-metadata-service
images:
  - /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - export
  - gridview
  - managed
  - metadata
  - sharepoint
  - terms
---
Migrating you Managed Metadata Terms to another SharePoint installation? First thing to do: Export.

This scripts exports all terms sorted by term group, term set and its level.

<!--more-->

Here's a sample export:

[![Exported Terms from Managed Metadata Service](/wp-content/uploads/2014/01/Exported-Terms-from-Managed-Metadata-Service.jpg)](/wp-content/uploads/2014/01/Exported-Terms-from-Managed-Metadata-Service.jpg)

And here's the script:

```powershell
<#
$Metadata = @{
	Title = "Export all Terms from Managed Metadata Service"
	Filename = "Export-ManagedMetadataServiceTerms.ps1"
	Description = ""
	Tags = "powershell, script, sharepoint, managed, metadata, terms, export"
	Project = ""
	Author = "Janik von Rotz"
	AuthorContact = "https://janikvonrotz.ch"
	CreateDate = "2014-01-14"
	LastEditDate = "2014-01-14"
	Url = ""
	Version = "0.0.0"
	License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit https://creativecommons.org/licenses/by-sa/3.0/ch/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#>

if((Get-PSSnapin 'Microsoft.SharePoint.PowerShell' -ErrorAction SilentlyContinue) -eq $null){Add-PSSnapin 'Microsoft.SharePoint.PowerShell'}

function loop{

    param(

        $Object,
        $AttributeName,
        $Level = 0
    )

    # check the child attribute containing the same type of objects
    $Objects = iex "`$Object.$AttributeName"

    # output this item
    $Object | select @{L="Object";E={$_}}, @{L="Level";E={$Level}}

    # output the child items of this object
    if($Objects){

        # add level
        $Level ++

        # loop trough the same function
        $Objects | %{loop -Object $_ -AttributeName $AttributeName -Level $Level}
    }
}

# reset vars
$SPTaxonomies = @()

# get all taxonomy objects
$SPTaxonomies = Get-SPTaxonomySession -Site "https://itwiki.vbl.ch" | %{

    $_.TermStores | %{

        $TermStore = New-Object -TypeName Psobject -Property @{

            TermStore = $_.Name
            Group = ""
            TermSet = ""
            Terms = ""
        }

        $_.Groups | %{

            $Group = $TermStore.PSObject.Copy()
            $Group.Group = $_.Name

            $_.TermSets | %{

                $TermSet = $Group.PSObject.Copy()
                $TermSet.TermSet = $_.Name
                $TermSet.Terms = ($_.Terms | %{loop -Object $_ -AttributeName "Terms" -Level 1})

                $TermSet
            }
        }
    }
}

# get maximum of levels a term has
$Levels = ($SPTaxonomies | %{$_.Terms | %{$_.Level}} | measure -Maximum).Maximum + 1

# loop throught term stores
$SPTaxonomies | %{

    $SPTaxonomy = $_

    # loop throught terms
    $_.Terms | %{

        # create a term export object
        $Item = $SPTaxonomy.PSObject.Copy()
        $Index = 1;while($Index -ne $Levels){
            $Item | Add-Member –MemberType NoteProperty –Name "Term Level $Index" –Value $(if($_.Level -eq $Index){$_.Object.Name}else{""})
            $Index += 1
        }

        # output this object
        $Item |  Select-Object * -exclude Terms
    }
} | Out-GridView
```

Latest version of this script: <a href="https://gist.github.com/8419585" target="_blank">https://gist.github.com/8419585</a>