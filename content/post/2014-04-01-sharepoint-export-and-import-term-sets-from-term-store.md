---
title: SharePoint Export and Import Term Sets from Term Store
date: 2014-04-01T07:34:23+00:00
author: Janik von Rotz
slug: sharepoint-export-and-import-term-sets-from-term-store
dsq_thread_id:
  - "2570434568"
image: /wp-content/uploads/2014/02/SharePoint-Online.jpg
categories:
  - SharePoint
tags:
  - csv
  - export
  - function
  - group
  - managed
  - metadata
  - powershell
  - powerup
  - script
  - sharepoint
  - term
---
The SharePoint term store service is one of the most important parts of a SharePoint installation. It's the best place to store an manage global metadata for your companies documents.

It's not that hard to design a [term set in Excel](http://technet.microsoft.com/en-us/library/ee424396(v=office.14).aspx) and import it afterwards with the built-in term store import wizard. 

However there's no tool or wizard that allows you to get a term set out of the term store. 

It's only possible with my PowerShell function (which is as always part of my [PowerShell PowerUp project](http://janikvonrotz.github.io/PowerShell-PowerUp/))
<!--more-->
```powershell
<#
$Metadata = @{
	Title = "Get SharePoint Managed Metadata Service Terms"
	Filename = "Get-SPManagedMetadataServiceTerms.ps1"
	Description = ""
	Tags = "powershell, function, sharepoint, managed, metadata, terms, export"
	Project = ""
	Author = "Janik von Rotz"
	AuthorContact = "https://janikvonrotz.ch"
	CreateDate = "2014-03-31"
	LastEditDate = "2014-03-31"
	Url = ""
	Version = "1.0.0"
	License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/ch/ or 
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#>

function Get-SPManagedMetadataServiceTerms{

<#
.SYNOPSIS
    List all Terms from Managed Metadata service.

.DESCRIPTION
	List all Terms form a specifified term store and optionally filter them by term group.

.PARAMETER Site
	Site running a taxonomy session.

.PARAMETER TermGroup
	Optionally filter the term group.

.EXAMPLE
	PS C:\> Get-SPManagedMetadataServiceTerms -Site "http://SharePoint.domain.com"

.EXAMPLE
	PS C:\> Get-SPManagedMetadataServiceTerms -Site "http://SharePoint.domain.com" -TermGroup "Wiki"

.LINK
    http://technet.microsoft.com/en-us/library/ee424396(v=office.14).aspx

#>

    [CmdletBinding()]
    param(

		[Parameter(Mandatory=$true)]
		[String]
		$Site,

		[String]
		$TermGroup
        
	)
    
    #--------------------------------------------------#
    # functions
    #--------------------------------------------------#

    function Add-ArrayLevelIndex{

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
            $Objects | %{Add-ArrayLevelIndex -Object $_ -AttributeName $AttributeName -Level $Level}

        }
    }

    #--------------------------------------------------#
    # modules
    #--------------------------------------------------# 
    if((Get-PSSnapin 'Microsoft.SharePoint.PowerShell' -ErrorAction SilentlyContinue) -eq $null){Add-PSSnapin 'Microsoft.SharePoint.PowerShell'}

    #--------------------------------------------------#
    # main
    #--------------------------------------------------#
    $SPTaxonomies = @()
    $TempTerm = New-Object -TypeName Psobject @{    
        Index = ""
        Term = ""
    }

    $i = 0;
    $TempTerms = while($i -ne 7){
        $i ++
        $e = $TempTerm.PSObject.Copy()
        $e.Index = $i
        $e
    }


    # get all taxonomy objects
    $SPTaxonomies = Get-SPTaxonomySession -Site $Site | %{

        $_.TermStores | %{
    
            $TermStore = New-Object -TypeName Psobject -Property @{
        
                TermStore = $_.Name
                "Term Group" = ""
                "Term Set Name" = ""
                "Term Set Description" = ""   
                LCID = ""             
                "Available for Tagging" = ""                
                Terms = ""      
            }        
        
        
            $_.Groups | Where{$_.Name -eq $TermGroup -or -not $TermGroup} | ForEach-Object{
            
                $Group = $TermStore.PSObject.Copy()
                $Group.'Term Group' = $_.Name                       
        
                $_.TermSets | %{
            
                    $TermSet = $Group.PSObject.Copy()
                    $TermSet.'Term Set Name' = $_.Name
                    $TermSet.'Term Set Description' = $_.Description
                                    
                    $TermSet.Terms = ($_.Terms | %{Add-ArrayLevelIndex -Object $_ -AttributeName "Terms" -Level 1})
            
                    $TermSet                  
                }        
            }        
        }
    }

    # get maximum of levels a term has
    # $Levels = ($SPTaxonomies | %{$_.Terms | %{$_.Level}} | measure -Maximum).Maximum + 1
    # or comment out and use default 7

    # loop throught term stores       
    $SPTaxonomies | %{

        $SPTaxonomy = $_

        # Output Termstore definitions
        $Item = $SPTaxonomy.PSObject.Copy()  
        $Item.'Available for Tagging' = if($_.IsAvailableForTagging){"TRUE"}else{"FALSE"}        
        $i = 0;while($i -ne 7){
            $i ++
            $Item | Add-Member –MemberType NoteProperty –Name "Level $i Term" –Value ""
        }
        $Item |  Select-Object 'Term Set Name','Term Set Description', LCID, 'Available for Tagging', 'Term Description', 'Level*'

    
        # loop throught terms
        $_.Terms | where{$_} | %{
                
            $Term = $_

            # create a term export object
            $Item = $SPTaxonomy.PSObject.Copy()   
             
            $Item.'Available for Tagging' = if($_.IsAvailableForTagging){"TRUE"}else{"FALSE"}
            $Item.'Term Set Name' = ""
            $Item.'Term Set Description' = ""
            
            $_.Object.Labels | ForEach-Object{

                $Item.LCID = $_.Language

                $Index = 0;while($Index -ne 7){
                    $Index ++

                    if($Term.Level -eq $Index){                        

                        $Item | Add-Member –MemberType NoteProperty –Name "Level $Index Term" –Value $Term.Object.Name

                        $TempTerms[$Index].Term = $Term.Object.Name

                    }elseif($Index -gt $Term.Level){
                            
                        $Item | Add-Member –MemberType NoteProperty –Name "Level $Index Term" –Value $Value

                    }elseif($Term.Level -gt $Index){

                        $Item | Add-Member –MemberType NoteProperty –Name "Level $Index Term" –Value $TempTerms[$Index].Term
                        
                    }                                   
                }  
            }   
        
            # output this object
            $Item |  Select-Object 'Term Set Name','Term Set Description', LCID, 'Available for Tagging', 'Term Description', 'Level*'
        }                
    }
}
```

With this function you can export the a specific term set into a CSV file or show it in the grid view.

	Get-SPManagedMetadataServiceTerms -Site "http://sharepoint.domain.com" -TermGroup "Wiki" | Export-Csv -Path TermSet.csv
	Get-SPManagedMetadataServiceTerms -Site "http://sharepoint.domain.com" -TermGroup "Wiki" | Out-GridView

Here's an example for an export to gridview.

[![Update Wiki Terms Excel from Termstore](/wp-content/uploads/2014/04/Update-Wiki-Terms-Excel-from-Termstore.gif)](https://janikvonrotz.ch/2014/04/01/sharepoint-export-and-import-term-sets-from-term-store/update-wiki-terms-excel-from-termstore/)