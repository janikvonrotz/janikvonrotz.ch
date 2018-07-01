---
id: 1789
title: Manage your SharePoint default settings with PowerShell
date: 2014-04-04T10:16:07+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=1789
permalink: /2014/04/04/manage-your-sharepoint-default-settings-with-powershell/
dsq_thread_id:
  - "2585449962"
image: /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - active
  - alternative
  - default
  - directory
  - display
  - groups
  - inheritance
  - language
  - name
  - navigation
  - powershell
  - script
  - settings
  - sharepoint
  - support
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

[code lang="ps"]
&lt;#
$Metadata = @{
	Title = &quot;SharePoint Default Settings&quot;
	Filename = &quot;Set-SPDefaultSettings.ps1&quot;
	Description = &quot;&quot;
	Tags = &quot;powershell, script, sharepoint, default settings&quot;
	Project = &quot;&quot;
	Author = &quot;Janik von Rotz&quot;
	AuthorContact = &quot;http://www.janikvonrotz.ch&quot;
	CreateDate = &quot;2013-05-07&quot;
	LastEditDate = &quot;2014-04-04&quot;
	Version = &quot;3.0.0&quot;
	License = @'
This work is licensed under the Creative Commons Attribution-NonCommercial-NoDerivs 3.0 Unported License.
To view a copy of this license, visit http://creativecommons.org/licenses/by-nc-nd/3.0/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#&gt;

try{

    #--------------------------------------------------#
    # modules
    #--------------------------------------------------#
    if((Get-PSSnapin &quot;Microsoft.SharePoint.PowerShell&quot; -ErrorAction SilentlyContinue) -eq $null){Add-PSSnapin &quot;Microsoft.SharePoint.PowerShell&quot;}
    Import-Module ActiveDirectory
    
    #--------------------------------------------------#
    # settings
    #--------------------------------------------------#
    
    $Configuration = @{
        SPSite = &quot;http://sharepoint.vbl.ch&quot;
        SPADGroupFilter = &quot;SP_*&quot;
        SPADGroupContainer = &quot;OU=SharePoint,OU=Services,OU=vblusers2,DC=vbl,DC=ch&quot;
        SPNavigationWebExclude = &quot;http://sharepoint.vbl.ch/Projekte&quot;
        AllowedVersioningTypes = &quot;Unternehmenswiki-Seite&quot;,&quot;Dokument&quot;,&quot;Wiki-Seite&quot;
        DisabledVersioningTypes = &quot;Survey&quot;
        SupportedLanguages = &quot;Deutsch&quot;, &quot;Englisch&quot;
    },
    @{
        SPSite = &quot;http://sharepoint.vbl.ch/itwiki&quot;
        SPADGroupFilter = &quot;SP_*&quot;
        SPADGroupContainer = &quot;OU=SharePoint,OU=Services,OU=vblusers2,DC=vbl,DC=ch&quot;
        AllowedVersioningTypes = &quot;Unternehmenswiki-Seite&quot;,&quot;Dokument&quot;,&quot;Wiki-Seite&quot;
        DisabledVersioningTypes = &quot;Survey&quot;
        SupportedLanguages = &quot;Deutsch&quot;, &quot;Englisch&quot;
    },
    @{
        SPSite = &quot;http://extranetvr.vbl.ch&quot;
        SPADGroupFilter = &quot;SP2_*&quot;
        SPADGroupContainer = &quot;OU=SharePoint,OU=Services,OU=vblusers2,DC=vbl,DC=ch&quot;
        AllowedVersioningTypes = &quot;Unternehmenswiki-Seite&quot;,&quot;Dokument&quot;,&quot;Wiki-Seite&quot;
        DisabledVersioningBaseTypes = &quot;Survey&quot;
        SupportedLanguages = &quot;Deutsch&quot;, &quot;Englisch&quot;
    }

    $Configuration | ForEach-Object{

        # get domain
        $ADDomain = ((Get-ADDomain).Name).ToUpper()
    
        # temp var for loops
        $Config = $_
            
            
        # update AD group names
              
        # SharePoint AD Groups 
        $ADGroups = Get-ADGroup -Filter * -SearchBase $_.SPADGroupContainer | Where-Object{$_.Name -like $Config.SPADGroupFilter}
            
        Get-SPUser -Limit All -Web $_.SPSite | Where-Object{$_.IsDomainGroup -and $_.Name -like &quot;$($ADDomain)\$Config.SPADGroupFilter&quot;} | ForEach-Object{
                
            $SPUser = $_
                    
            $ADGroups | Where-Object{
                
                # without claims
                (($_.SID -eq $SPUser.Sid) -or 
                                
                # claims
                ($SPUser.LoginName -like &quot;*$($_.SID)&quot;)) -and 

                # check name
                (&quot;$ADDomain\$(($_.Name).ToLower())&quot; -ne $SPUser.Name.ToLower())
                
            } | ForEach-Object{
            
                Write-PPEventLog -Message &quot;Change Displayname for SPGroup: $($SPUser.Name) to: &quot; + &quot;$ADDomain\$(($_.Name).ToLower())&quot; -Source &quot;SharePoint Default Settings&quot; -WriteMessage        
                Set-SPUser $SPUser -DisplayName &quot;$ADDomain\$(($_.Name).ToLower())&quot;
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
            
                    Write-PPEventLog -Message &quot;Enable Versioning for: $($_.title) on: $($_.parentweb.title).&quot; -Source &quot;SharePoint Default Settings&quot; -WriteMessage
                        
                    $_.EnableVersioning = $true
                    $_.MajorVersionLimit = 10   
                    $_.Update()   
                
                # disable versioning fore everything else
                }elseif(($_.EnableVersioning -eq $true) -and -not ($Types | where{$Config.AllowedVersioningTypes -contains $_})){
            
                    Write-PPEventLog -Message &quot;Disable Versioning for: $($_.title) on: $($_.parentweb.title).&quot; -Source &quot;SharePoint Default Settings&quot; -WriteMessage
            
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
                
                Write-PPEventLog -Message &quot;Enable language: $($_.DisplayName) for SPWeb: $($SPWeb.Url).&quot; -Source &quot;SharePoint Default Settings&quot; -WriteMessage
                $CultureInfo = New-Object System.Globalization.CultureInfo($_.LCID)
                $SPWeb.AddSupportedUICulture($CultureInfo)
            }            

            $SPWeb.Update()
        }  
    }
}catch{

    Write-PPErrorEventLog -Source &quot;SharePoint Default Settings&quot; -ClearErrorVariable
}
[/code]

Latest version of this script: [https://gist.github.com/7871902](https://gist.github.com/7871902)
