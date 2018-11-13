---
title: Manage your SharePoint default settings with PowerShell
date: 2014-04-04T10:16:07+00:00
author: Janik von Rotz
slug: manage-your-sharepoint-default-settings-with-powershell
images:
  - /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - powershell
  - script
  - sharepoint
  - versioning
---
*This post is part of my [PowerShell PowerUp](https://github.com/janikvonrotz/PowerShell-PowerUp) project.*

The only way to deploy SharePoint default settings for site collections and sites manually is using predefined site templates or an ugly third party tool.
And in case you'll do that with site templates it'll cost a lot of the time to update settings which were already set.

However with PowerShell it's possible to deploy SharePoint default settings automatically and it's a lot easier than you might think.
<!--more-->
Today I want to show you my SharePoint default settings script. It does a lot of work for me:

* Update Active Directory group display names
* Set the navigation inheritance
* Set a corporate site logo
* Enable and disable versioning on selected lists and libraries
* Add support for multilingualism globally
* Enable selected installed languages on websites

There are some functions in the script which you can't know f.g. `Write-PPEventLog`, these are custom functions and are part of my [PowerShell PowerUp](https://github.com/janikvonrotz/PowerShell-PowerUp) project.

```powershell
<#
$Metadata = @{
	Title = "SharePoint Default Settings"
	Filename = "Set-SPDefaultSettings.ps1"
	Description = ""
	Tags = "powershell, script, sharepoint, default settings"
	Project = ""
	Author = "Janik von Rotz"
	AuthorContact = "http://www.janikvonrotz.ch"
	CreateDate = "2013-05-07"
	LastEditDate = "2014-04-04"
	Version = "3.0.0"
	License = @'
This work is licensed under the Creative Commons Attribution-NonCommercial-NoDerivs 3.0 Unported License.
To view a copy of this license, visit http://creativecommons.org/licenses/by-nc-nd/3.0/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#>

try{

    #--------------------------------------------------#
    # modules
    #--------------------------------------------------#
    if((Get-PSSnapin "Microsoft.SharePoint.PowerShell" -ErrorAction SilentlyContinue) -eq $null){Add-PSSnapin "Microsoft.SharePoint.PowerShell"}
    Import-Module ActiveDirectory
    
    #--------------------------------------------------#
    # settings
    #--------------------------------------------------#
    
    $Configuration = @{
        SPSite = "http://sharepoint.vbl.ch"
        SPADGroupFilter = "SP_*"
        SPADGroupContainer = "OU=SharePoint,OU=Services,OU=vblusers2,DC=vbl,DC=ch"
        SPNavigationWebExclude = "http://sharepoint.vbl.ch/Projekte"
        AllowedVersioningTypes = "Unternehmenswiki-Seite","Dokument","Wiki-Seite"
        DisabledVersioningTypes = "Survey"
        SupportedLanguages = "Deutsch", "Englisch"
    },
    @{
        SPSite = "http://sharepoint.vbl.ch/itwiki"
        SPADGroupFilter = "SP_*"
        SPADGroupContainer = "OU=SharePoint,OU=Services,OU=vblusers2,DC=vbl,DC=ch"
        AllowedVersioningTypes = "Unternehmenswiki-Seite","Dokument","Wiki-Seite"
        DisabledVersioningTypes = "Survey"
        SupportedLanguages = "Deutsch", "Englisch"
    },
    @{
        SPSite = "http://extranetvr.vbl.ch"
        SPADGroupFilter = "SP2_*"
        SPADGroupContainer = "OU=SharePoint,OU=Services,OU=vblusers2,DC=vbl,DC=ch"
        AllowedVersioningTypes = "Unternehmenswiki-Seite","Dokument","Wiki-Seite"
        DisabledVersioningBaseTypes = "Survey"
        SupportedLanguages = "Deutsch", "Englisch"
    }

    $Configuration | ForEach-Object{

        # get domain
        $ADDomain = ((Get-ADDomain).Name).ToUpper()
    
        # temp var for loops
        $Config = $_
            
            
        # update AD group names
              
        # SharePoint AD Groups 
        $ADGroups = Get-ADGroup -Filter * -SearchBase $_.SPADGroupContainer | Where-Object{$_.Name -like $Config.SPADGroupFilter}
            
        Get-SPUser -Limit All -Web $_.SPSite | Where-Object{$_.IsDomainGroup -and $_.Name -like "$($ADDomain)\$Config.SPADGroupFilter"} | ForEach-Object{
                
            $SPUser = $_
                    
            $ADGroups | Where-Object{
                
                # without claims
                (($_.SID -eq $SPUser.Sid) -or 
                                
                # claims
                ($SPUser.LoginName -like "*$($_.SID)")) -and 

                # check name
                ("$ADDomain\$(($_.Name).ToLower())" -ne $SPUser.Name.ToLower())
                
            } | ForEach-Object{
            
                Write-PPEventLog -Message "Change Displayname for SPGroup: $($SPUser.Name) to: " + "$ADDomain\$(($_.Name).ToLower())" -Source "SharePoint Default Settings" -WriteMessage        
                Set-SPUser $SPUser -DisplayName "$ADDomain\$(($_.Name).ToLower())"
            } 
        }       
         
        
        # website settings

        Get-SPSite | where{$_.Url -eq $Config.SPSite} | Get-SPWeb -Limit All  | ForEach-Object{

            # temp var for loops
            $SPWeb = $_


            # update navigation inheritance except for excluded sites

    	    if(
            ($_.Url).startsWith($Config.SPSite) -and     
            ($_.Url -ne $Config.SPSite) -and    
            ($Config.SPNavigationWebExclude -notcontains $_.Url)
            ){            
    		    $SPPublishingWeb = [Microsoft.SharePoint.Publishing.PublishingWeb]::GetPublishingWeb($_)
    		    $SPPublishingWeb.Navigation.InheritGlobal = $true
    		    $SPPublishingWeb.Navigation.GlobalIncludeSubSites = $true
    		    $SPPublishingWeb.Update()
            }
        

            # update Site Logo

            $_.SiteLogoUrl = $(Get-SPWeb $Config.SPSite).SiteLogoUrl
            $_.Update()
        

            # Enable versioning on lists

            $_.Lists | Where-Object{ $Config.DisabledVersioningBaseTypes -notcontains $_.basetype} | ForEach-Object{
            
                # get content types foreach list
                $Types = $_.ContentTypes | %{$_.Name}
            
                # enable versionging for document libraries and wiki sites
                if(($Types | where{$Config.AllowedVersioningTypes -contains $_}) -and ($_.EnableVersioning -eq $false)){
            
                    Write-PPEventLog -Message "Enable Versioning for: $($_.title) on: $($_.parentweb.title)." -Source "SharePoint Default Settings" -WriteMessage
                        
                    $_.EnableVersioning = $true
                    $_.MajorVersionLimit = 10   
                    $_.Update()   
                
                # disable versioning fore everything else
                }elseif(($_.EnableVersioning -eq $true) -and -not ($Types | where{$Config.AllowedVersioningTypes -contains $_})){
            
                    Write-PPEventLog -Message "Disable Versioning for: $($_.title) on: $($_.parentweb.title)." -Source "SharePoint Default Settings" -WriteMessage
            
                    $_.EnableVersioning = $false       
                    $_.Update()
                }
            }


            # Enable alternative languages

            # enable multilingualism globally
            $_.IsMultilingual = $true

            # get already enabled languages IDs
            $EnabledLanguageIDs = $_.SupportedUICultures | ForEach-Object{$_.LCID}

            # get regtional settings objects
            $SPRegionalSettings = New-Object Microsoft.SharePoint.SPRegionalSettings($_)

            # enable every installed and supported language as longs it's not yet enabled
            $SPRegionalSettings.InstalledLanguages | where{$Config.SupportedLanguages -contains $_.DisplayName -and $EnabledLanguageIDs -notcontains $_.LCID} | ForEach-Object{
                
                Write-PPEventLog -Message "Enable language: $($_.DisplayName) for SPWeb: $($SPWeb.Url)." -Source "SharePoint Default Settings" -WriteMessage
                $CultureInfo = New-Object System.Globalization.CultureInfo($_.LCID)
                $SPWeb.AddSupportedUICulture($CultureInfo)
            }            

            $SPWeb.Update()
        }  
    }
}catch{

    Write-PPErrorEventLog -Source "SharePoint Default Settings" -ClearErrorVariable
}
```

Latest version of this script: [https://gist.github.com/7871902](https://gist.github.com/7871902)
