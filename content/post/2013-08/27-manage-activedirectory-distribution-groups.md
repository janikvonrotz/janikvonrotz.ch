---
title: Manage ActiveDirectory Distribution Groups
date: 2013-08-27T12:39:11+00:00
author: Janik Vonrotz
slug: manage-activedirectory-distribution-groups
images:
  - /wp-content/uploads/2013/07/PowerShell.png
categories:
  - Office 365
  - scripting
tags:
  - adfs
  - distribution
  - exchange
  - office365
  - powershell
  - project
  - scripting
  - snippet
---
With Office365 connected with an ADFS you have to redesgin your Exchange distribution groups. ADFS only syncs distribution groups that have these definitions:

<ul>
    <li><del>Group scope is universal</del></li>
    <li>Group type is distribution</li>
    <li>Group members have to be users
<ul>
    <li>Yes, it's not possible to have security groups or something else as distribution group members.</li>
</ul>
</li>
</ul>

My idea was simple, I'm developing a script that creates for every OU and child OU I'm chosing in the ActiveDirectory structure a distribution list containing the users of the chosen OU recursively.

<!--more-->

While developing this I've added some cool features, in addition you can:

<ul>
    <li>Copy the members of one or more groups into another.</li>
    <li>Remove the members of one or more group in an another group.</li>
    <li>Exclude users in distribution groups.</li>
    <li>Exclude distribution groups.</li>
</ul>

By default the script will only add enabled users with an email address.

This script makes use of the <a href="https://github.com/janikvonrotz/Powershell-Profile">PowerShell Profile</a> environment, f.e. the function `Send-PPErrorReport` sends an error report per email when the script fails or produces problems.

```powershell
<#
$Metadata = @{
    Title = "New ActiveDirectory Distribution Groups"
    Filename = "New-ADDistributionGroups.ps1"
    Description = "Create or update ActiveDirectory distribution groups"
    Tags = "powershell, activedirectory, distribution, groups, create, update"
    Project = ""
    Author = "Janik Vonrotz"
    AuthorContact = "https://janikvonrotz.ch"
    CreateDate = "2013-08-27"
    LastEditDate = "2013-09-30"
    Url = "https://gist.github.com/6352037"
    Version = "1.1.0"
    License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit https://creativecommons.org/licenses/by-sa/3.0/ch/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#>

#--------------------------------------------------#
# modules
#--------------------------------------------------#
Import-Module ActiveDirectory

# set OUs where the distributions groups should be enabled
$OUs = @(
    @{Name = "OU=Betrieb,OU=vblusers2,DC=vbl,DC=ch"},
    @{Name = "OU=Direktion,OU=vblusers2,DC=vbl,DC=ch"},
    @{Name = "OU=Finanzen,OU=vblusers2,DC=vbl,DC=ch"},
    @{Name = "OU=Personal,OU=vblusers2,DC=vbl,DC=ch"},
    @{Name = "OU=Technik,OU=vblusers2,DC=vbl,DC=ch"}
)

# list of users to exclude in distribution groups
$ExcludeUsers = "abascan","ba test","ba-service"

# list of distribution groups to exclude
$ExcludeOUs = "Verwaltungsrat"

# special configuration to handle special
$Configs = @(
    @{
        Name = "GL";
        Options = @("UpdateFromGroups");
        AddGroups = @("Geschäftsleitung Gruppe")
    },
    @{
        Name = "GL erw";
        Options = @("UpdateFromGroups");
        AddGroups = @("Erweiterte Geschäftsleitung Gruppe")
    },
    @{
        Name = "Alle";
        Options = @("UpdateFromGroups");
        AddGroups = @("Technik","Betrieb","Personal","Finanzen","GL","Kommunikation","Sekretariat")
    },
    @{
        Name = "Alle mit Arbeitsplatz";
        Options = @("UpdateFromGroups","RemoveGroups");
        AddGroups = @("Alle");
        RemoveGroups = @("SPO_365E1License")
    },
    @{
        Name = "Alle ohne Arbeitsplatz";
        Options = @("UpdateFromGroups");
        AddGroups = @("SPO_365E1License");
    },
    @{
        Name = "Fahrdienst A - Hermann M";
        Options = @("UpdateFromGroups");
        AddGroups = @("Fahrdienst A - Hermann M Gruppe");
    },
    @{
        Name = "Fahrdienst A - Segui M";
        Options = @("UpdateFromGroups");
        AddGroups = @("Fahrdienst A - Segui M Gruppe");
    },
    @{
        Name = "Fahrdienst B - Nietlispach M";
        Options = @("UpdateFromGroups");
        AddGroups = @("Fahrdienst B - Nietlispach M Gruppe");
    },
    @{
        Name = "Fahrdienst B - Zaugg D";
        Options = @("UpdateFromGroups");
        AddGroups = @("Fahrdienst B - Zaugg D Gruppe");
    },
    @{
        Name = "Fahrdienst C - Habegger R";
        Options = @("UpdateFromGroups");
        AddGroups = @("Fahrdienst C - Habegger R Gruppe");
    },
    @{
        Name = "Fahrdienst C - Malbasic N";
        Options = @("UpdateFromGroups");
        AddGroups = @("Fahrdienst C - Malbasic N Gruppe");
    },
    @{
        Name = "Fahrdienst D - Küchler P";
        Options = @("UpdateFromGroups");
        AddGroups = @("Fahrdienst D - Küchler P Gruppe");
    },
    @{
        Name = "Fahrdienst D - Zimmermann L";
        Options = @("UpdateFromGroups");
        AddGroups = @("Fahrdienst D - Zimmermann L Gruppe");
    },
    @{
        Name = "Fahrdienst E - Bechter K";
        Options = @("UpdateFromGroups");
        AddGroups = @("Fahrdienst E - Bechter K Gruppe");
    },
    @{
        Name = "Fahrdienst E - Brunner R";
        Options = @("UpdateFromGroups");
        AddGroups = @("Fahrdienst E - Brunner R Gruppe");
    },
    @{
        Name = "Fahrdienst F - Bieri René";
        Options = @("UpdateFromGroups");
        AddGroups = @("Fahrdienst F - Bieri René Gruppe");
    },
    @{
        Name = "Fahrdienst F - Bieri Urs";
        Options = @("UpdateFromGroups");
        AddGroups = @("Fahrdienst F - Bieri Urs Gruppe");
    },
    @{
        Name = "Verkehrsdisponnenten";
        Options = @("UpdateFromGroups");
        AddGroups = @("F_Verkehrsdisponnenten");
    },
    @{
        Name = "Personalkommission";
        Options = @("UpdateFromGroups");
        AddGroups = @("Personalkommission Abteilung");
    }
)

# get all OUs recursive
$OUs = $OUs | %{Get-ADOrganizationalUnit -Filter "*" -SearchBase $_.Name} | where {-not ($ExcludeOUs -contains $_.Name)}

# check in every OU if a distribution group with the same name as the OU exist
$OUs | %{$OU = $_.DistinguishedName;
    if(Get-ADGroup -Filter {SamAccountName -eq $_.Name -and GroupCategory -eq "Distribution"} | Where-Object{$_.DistinguishedName -like "*$OU"}){

        Write-Host "Update users in distribution group $($_.Name)."
        $ADGroup = Get-ADGroup -Filter {SamAccountName -eq $_.Name -and GroupCategory -eq "Distribution"}
        Get-ADGroupMember -Identity $ADGroup | %{Remove-ADGroupMember -Identity $ADGroup -Members $_ -Confirm:$false}
        Get-ADUser -Filter {EmailAddress -like "*"} -SearchBase $OU | where {$_.enabled -eq $true -and -not ($ExcludeUsers -contains $_.Name)} | where{$_ -ne $null} | %{Add-ADGroupMember -Identity $ADGroup -Members $_}

    }else{

        Write-Host "Create distribution group $($_.Name)."
        New-ADGroup -Name $_.Name -SamAccountName $_.Name -GroupCategory Distribution -GroupScope Universal -DisplayName $_.Name -Path $($_.DistinguishedName) -Description "Distribution group for $($_.Name)."
        $ADGroup = Get-ADGroup $_.Name
        Get-ADUser -Filter {EmailAddress -like "*"} -SearchBase $_.DistinguishedName | where {$_.enabled -eq $true -and -not ($ExcludeUsers -contains $_.Name)} | where{$_ -ne $null} | %{Add-ADGroupMember -Identity $ADGroup -Members $_}
    }
}

# custom configuration
$Configs | %{

    $ADGroup = Get-ADGroup -Identity $_.Name
    $Config = $_

    if($_.Options -match "UpdateFromGroups"){

        Write-Host "Add users from $($Config.AddGroups) to $($ADGroup.Name)."
        Get-ADGroupMember -Identity $ADGroup | %{Remove-ADGroupMember -Identity $ADGroup -Members $_ -Confirm:$false}
        $Config.AddGroups | %{Get-ADGroupMember -Identity $_ -Recursive | Get-ADUser | where {($_.enabled -eq $true) -and -not ($ExcludeUsers -contains $_.Name)}} | select -Unique | %{Add-ADGroupMember -Identity $ADGroup -Members $_}

    }

    if($_.Options -match "RemoveGroups"){

        Write-Host "Remove users from $($Config.RemoveGroups) in $($ADGroup.Name)."
        $ADGroupMembers = Get-ADGroupMember -Identity $ADGroup
        $Config.RemoveGroups | %{Get-ADGroupMember -Identity $_ -Recursive | Get-ADUser | where {($ADGroupMembers -match $_) -and ($_.enabled -eq $true) -and -not ($ExcludeUsers -contains $_.Name)}} | select -Unique  | %{Remove-ADGroupMember -Identity $ADGroup -Members $_ -Confirm:$false}

    }
}

if($error){
    Send-PPErrorReport -FileName "activedirectory.mail.config.xml" -ScriptName $MyInvocation.InvocationName
}
```

Latest version of this script: <a href="https://gist.github.com/6352037">https://gist.github.com/6352037</a>