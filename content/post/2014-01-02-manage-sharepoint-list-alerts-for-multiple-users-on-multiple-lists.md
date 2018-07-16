---
title: manage SharePoint list alerts for multiple users on multiple lists
date: 2014-01-02T14:38:25+00:00
author: Janik von Rotz
slug: manage-sharepoint-list-alerts-for-multiple-users-on-multiple-lists
dsq_thread_id:
  - "2087265617"
image: /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - PowerShell
  - SharePoint
tags:
  - alert
  - deploy
  - manage
  - multiple
  - powershell
  - script
  - sharepoint
  - users
  - worfklow
---
In certain cases SharePoint alerts are more useful than workflows, f.e. having the possibility to let users manager their alerts is not possible with workflows.

To manager alerts for a couple of users on specific lists I've written a script that handles the whole cycle of alerts. It let me delete, update and create alerts based on predefined configuration.

<!--more-->

```powershell
<#
$Metadata = @{
	Title = "Update SharePoint User Alerts"
	Filename = "Update-SPUserAlerts.ps1"
	Description = ""
	Tags = "powershell, sharepoint, update, user, alerts"
	Project = ""
	Author = "Janik von Rotz"
	AuthorContact = "https://janikvonrotz.ch"
	CreateDate = "2014-01-02"
	LastEditDate = "2014-01-02"
	Url = ""
	Version = "1.0.0"
	License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/3.0/ch/ or 
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#>

try{

    #--------------------------------------------------#
    # modules
    #--------------------------------------------------#
    if((Get-PSSnapin 'Microsoft.SharePoint.PowerShell' -ErrorAction SilentlyContinue) -eq $null){Add-PSSnapin 'Microsoft.SharePoint.PowerShell'}
    Import-Module ActiveDirectory
    
    #--------------------------------------------------#
    # settings
    #--------------------------------------------------#
    $Alerts = @(    
        @{    
            ID = 1
            ListUrl = "http://sharepoint.domain.ch/site/subsite/Lists/ListName/view.aspx"    
            SubscriberADUsersAndGroups = "ADGroup","ADUser"          
            Title = "`"Benachrichtigunggg `$Username`""
            AlertType = [Microsoft.SharePoint.SPAlertType]::List       
            DeliveryChannels = [Microsoft.SharePoint.SPAlertDeliveryChannels]::Email
            EventType = [Microsoft.SharePoint.SPEventType]::Add
            AlertFrequency = [Microsoft.SharePoint.SPAlertFrequency]::Immediate
            ListViewName = "View 2014"
            FilterIndex = 8
        }
    )
    
    #--------------------------------------------------#
    # function
    #--------------------------------------------------#
    function New-UnifiedAlertObject{
        
        param(
            $Title,
            $AlertType,
            $DeliveryChannels,
            $EventType,
            $AlertFrequency,
            $ListViewID,
            $FilterIndex
        )
        
        New-Object PSObject -Property @{
            Title = $Title
            AlertType = $AlertType
            DeliveryChannels = $DeliveryChannels
            EventType = $EventType
            AlertFrequency = $AlertFrequency
            ListViewID = $ListViewID
            AlertFilterIndex = $AlertFilterIndex
        }
    }

    #--------------------------------------------------#
    # main
    #--------------------------------------------------#
    
    $Alerts | %{
    
        # set vars
        $Message = "Update alerts with ID: $($_.ID)`n"
        $Alert = $_
    
        # get sp site
        $SPWeb = Get-SPWeb (Get-SPUrl $_.ListUrl).WebUrl

        # get name of the list
        $ListName = (Get-SPUrl $_.ListUrl).Url -replace ".*/",""

        # get the sp list object
        $SPList = $SPWeb.Lists[$ListName]
        
        $SPUsers = Get-SPUser -Web $SPWeb.Site.Url
        
        # get the id of the list view by name
        $SPListViewID = ($SPList.Views | where{$_.title -eq $SPListViewName -and $_.title -ne ""} | select -First 1).ID
        
        # get existing alerts
        $ExistingAlerts = $SPWeb.Alerts | where{$_.Properties["alertid"] -eq $Alert.ID}
        
        # cycle throught all users and  update, create or delete their alerts
        $UserWithAlerts = $_.SubscriberADUsersAndGroups | %{
        
            Get-ADObject -Filter {Name -eq $_} | %{
        
                if($_.ObjectClass -eq "user"){
                    
                    Get-ADUser $_.DistinguishedName
                    
                }elseif($_.ObjectClass -eq "group"){
                
                    Get-ADGroupMember $_.DistinguishedName -Recursive | Get-ADUser 
                }
            } | %{
                
                $ADUser = $_
            
                $SPUsers | where{$_.SID -eq $ADUser.SID} | %{$SPWeb.EnsureUser($_.Name)}
            }
        } | %{
        
            # create alert title
            $Username = $_.DisplayName
            $AlertTitle = Invoke-Command -ScriptBlock ([ScriptBlock]::Create($Alert.Title))
            
            # check if already alert exists with this id
            $AlertIS = $_.Alerts | where{$_.Properties["alertid"] -eq $Alert.ID} | select -First 1           
            
            # if exists update this alert
            if($AlertIS){
                
                # create alert objects to compare
                $AlertObjectIS = New-UnifiedAlertObject -Title $AlertIS.title `
                    -AlertType $AlertIS.AlertType `
                    -DeliveryChannels $AlertIS.DeliveryChannels `
                    -EventType $AlertIS.EventType `
                    -AlertFrequency $AlertIS.AlertFrequency `
                    -ListViewID $AlertIS.Properties["filterindex"] `
                    -FilterIndex $AlertIS.Properties["filterindex"]
                    
                $AlertObjectTo = New-UnifiedAlertObject -Title $AlertTitle `
                    -AlertType $Alert.AlertType `
                    -DeliveryChannels $Alert.DeliveryChannels `
                    -EventType $Alert.EventType `
                    -AlertFrequency $Alert.AlertFrequency `
                    -ListViewID $SPListViewID `
                    -FilterIndex $Alert.FilterIndex
                
                # only update changed attributes
                if(Compare-Object -ReferenceObject $AlertObjectTo -DifferenceObject $AlertObjectIS -Property Title, AlertType, DeliveryChannels, EventType, AlertFrequency, ListViewID, FilterIndex){
                    
                    $Message += "Update alert with ID: $($Alert.ID) for user: $($_.DisplayName)`n"
                    
                    if(Compare-Object -ReferenceObject $AlertObjectTo -DifferenceObject $AlertObjectIS -Property Title){$AlertIS.Title = $AlertObjectTo.Title}
                    if(Compare-Object -ReferenceObject $AlertObjectTo -DifferenceObject $AlertObjectIS -Property AlertType){$AlertIS.AlertType = $AlertObjectTo.AlertType}
                    if(Compare-Object -ReferenceObject $AlertObjectTo -DifferenceObject $AlertObjectIS -Property DeliveryChannels){$AlertIS.DeliveryChannels = $AlertObjectTo.DeliveryChannels}
                    if(Compare-Object -ReferenceObject $AlertObjectTo -DifferenceObject $AlertObjectIS -Property EventType){$AlertIS.EventType = $AlertObjectTo.EventType}
                    if(Compare-Object -ReferenceObject $AlertObjectTo -DifferenceObject $AlertObjectIS -Property AlertFrequency){$AlertIS.AlertFrequency = $AlertObjectTo.AlertFrequency}
                    if(Compare-Object -ReferenceObject $AlertObjectTo -DifferenceObject $AlertObjectIS -Property ListViewID){$AlertIS.Properties["viewid"] = $AlertObjectTo.ListViewID}
                    if(Compare-Object -ReferenceObject $AlertObjectTo -DifferenceObject $AlertObjectIS -Property FilterIndex){$AlertIS.Properties["filterindex"] = $AlertObjectTo.FilterIndex}
                    
                    # update changes
                    $AlertIS.Update()
                }
            }else{
            
                # create a new alert object  
                $Message += "Create alert with ID: $($Alert.ID) for user: $($_.DisplayName)`n"          
                $NewAlert = $_.Alerts.Add()
                
                # add attributes
                $NewAlert.Properties.Add("alertid",$Alert.ID)
                $NewAlert.Title = $AlertTitle                  
                if($SPListViewID){                
                    $NewAlert.Properties.Add("filterindex",$Alert.FilterIndex)
                    $NewAlert.Properties.Add("viewid",$SPListViewID)              
                }
                $NewAlert.AlertType = $Alert.AlertType
                $NewAlert.List = $SPList
                $NewAlert.DeliveryChannels = $Alert.DeliveryChannels
                $NewAlert.EventType = $Alert.EventType
                $NewAlert.AlertFrequency = $Alert.AlertFrequency

                # create the alert
                $NewAlert.Update()
            }
            
            # pipe the users to check alerts to delete
            $_
            
        } 
        
        # username array
        $UserWithAlerts = $UserWithAlerts | %{"$($_.UserLogin)"}
        
        # delete alerts
        $ExistingAlerts | where{$UserWithAlerts -notcontains $_.User} | %{
        
            $Message += "Delete alert with ID: $($Alert.ID) for user: $($_.User)`n"
            $SPWeb.Alerts.Delete($_.ID)        
        }
        
        Write-PPEventLog -Message $Message -Source "Update SharePoint User Alerts" -WriteMessage
    }
}catch{

    Write-PPErrorEventLog -Source "Update SharePoint User Alerts" -ClearErrorVariable    
}
```

The configuration attribute ListViewName and FilterIndex allows me to change to create an alert for an specific view.

![SharePoint Create Alert on List View](/wp-content/uploads/2014/01/SharePoint-Create-Alert-on-List-View-746x1024.jpg)

<h1>Requirements</h1>

To use this script I'll recommend you to install my project: <a href="https://github.com/janikvonrotz/PowerShell-Profile" target="_blank">PowerShell </a>Profile. There are some function as f.e. Get-SPUrl which are not defined in this script but are part of this project.

You'll find the latest version of this script here: <a href="https://gist.github.com/janikvonrotz/8218907" title="https://gist.github.com/janikvonrotz/8218907">https://gist.github.com/janikvonrotz/8218907</a>