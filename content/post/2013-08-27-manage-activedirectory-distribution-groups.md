---
id: 464
title: Manage ActiveDirectory Distribution Groups
date: 2013-08-27T12:39:11+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=464
permalink: /2013/08/27/manage-activedirectory-distribution-groups/
dsq_thread_id:
  - "1654234472"
image: /wp-content/uploads/2013/07/PowerShell.png
categories:
  - Active Directory
  - Blog
  - Exchange
  - Office 365
  - PowerShell
tags:
  - adfs
  - distribution
  - exchange
  - groups
  - office365
  - powershell
  - profile
  - project
  - redesign
  - script
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

This script makes use of the <a href="https://github.com/janikvonrotz/Powershell-Profile">PowerShell Profile</a> environment, f.e. the function <code>Send-PPErrorReport</code> sends an error report per email when the script fails or produces problems.

[code lang="ps"]
&lt;#
$Metadata = @{
    Title = &quot;New ActiveDirectory Distribution Groups&quot;
    Filename = &quot;New-ADDistributionGroups.ps1&quot;
    Description = &quot;Create or update ActiveDirectory distribution groups&quot;
    Tags = &quot;powershell, activedirectory, distribution, groups, create, update&quot;
    Project = &quot;&quot;
    Author = &quot;Janik von Rotz&quot;
    AuthorContact = &quot;https://janikvonrotz.ch&quot;
    CreateDate = &quot;2013-08-27&quot;
    LastEditDate = &quot;2013-09-30&quot;
    Url = &quot;https://gist.github.com/6352037&quot;
    Version = &quot;1.1.0&quot;
    License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit https://creativecommons.org/licenses/by-sa/3.0/ch/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#&gt;

#--------------------------------------------------#
# modules
#--------------------------------------------------#
Import-Module ActiveDirectory

# set OUs where the distributions groups should be enabled
$OUs = @(
    @{Name = &quot;OU=Betrieb,OU=vblusers2,DC=vbl,DC=ch&quot;},
    @{Name = &quot;OU=Direktion,OU=vblusers2,DC=vbl,DC=ch&quot;},
    @{Name = &quot;OU=Finanzen,OU=vblusers2,DC=vbl,DC=ch&quot;},
    @{Name = &quot;OU=Personal,OU=vblusers2,DC=vbl,DC=ch&quot;},
    @{Name = &quot;OU=Technik,OU=vblusers2,DC=vbl,DC=ch&quot;}
)

# list of users to exclude in distribution groups
$ExcludeUsers = &quot;abascan&quot;,&quot;ba test&quot;,&quot;ba-service&quot;

# list of distribution groups to exclude
$ExcludeOUs = &quot;Verwaltungsrat&quot;

# special configuration to handle special
$Configs = @(
    @{
        Name = &quot;GL&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;Geschäftsleitung Gruppe&quot;)
    },
    @{
        Name = &quot;GL erw&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;Erweiterte Geschäftsleitung Gruppe&quot;)
    },
    @{
        Name = &quot;Alle&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;Technik&quot;,&quot;Betrieb&quot;,&quot;Personal&quot;,&quot;Finanzen&quot;,&quot;GL&quot;,&quot;Kommunikation&quot;,&quot;Sekretariat&quot;)
    },
    @{
        Name = &quot;Alle mit Arbeitsplatz&quot;;
        Options = @(&quot;UpdateFromGroups&quot;,&quot;RemoveGroups&quot;);
        AddGroups = @(&quot;Alle&quot;);
        RemoveGroups = @(&quot;SPO_365E1License&quot;)
    },
    @{
        Name = &quot;Alle ohne Arbeitsplatz&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;SPO_365E1License&quot;);
    },
    @{
        Name = &quot;Fahrdienst A - Hermann M&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;Fahrdienst A - Hermann M Gruppe&quot;);
    },
    @{
        Name = &quot;Fahrdienst A - Segui M&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;Fahrdienst A - Segui M Gruppe&quot;);
    },
    @{
        Name = &quot;Fahrdienst B - Nietlispach M&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;Fahrdienst B - Nietlispach M Gruppe&quot;);
    },
    @{
        Name = &quot;Fahrdienst B - Zaugg D&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;Fahrdienst B - Zaugg D Gruppe&quot;);
    },
    @{
        Name = &quot;Fahrdienst C - Habegger R&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;Fahrdienst C - Habegger R Gruppe&quot;);
    },
    @{
        Name = &quot;Fahrdienst C - Malbasic N&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;Fahrdienst C - Malbasic N Gruppe&quot;);
    },
    @{
        Name = &quot;Fahrdienst D - Küchler P&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;Fahrdienst D - Küchler P Gruppe&quot;);
    },
    @{
        Name = &quot;Fahrdienst D - Zimmermann L&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;Fahrdienst D - Zimmermann L Gruppe&quot;);
    },
    @{
        Name = &quot;Fahrdienst E - Bechter K&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;Fahrdienst E - Bechter K Gruppe&quot;);
    },
    @{
        Name = &quot;Fahrdienst E - Brunner R&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;Fahrdienst E - Brunner R Gruppe&quot;);
    },
    @{
        Name = &quot;Fahrdienst F - Bieri René&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;Fahrdienst F - Bieri René Gruppe&quot;);
    },
    @{
        Name = &quot;Fahrdienst F - Bieri Urs&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;Fahrdienst F - Bieri Urs Gruppe&quot;);
    },
    @{
        Name = &quot;Verkehrsdisponnenten&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;F_Verkehrsdisponnenten&quot;);
    },
    @{
        Name = &quot;Personalkommission&quot;;
        Options = @(&quot;UpdateFromGroups&quot;);
        AddGroups = @(&quot;Personalkommission Abteilung&quot;);
    }
)

# get all OUs recursive
$OUs = $OUs | %{Get-ADOrganizationalUnit -Filter &quot;*&quot; -SearchBase $_.Name} | where {-not ($ExcludeOUs -contains $_.Name)}

# check in every OU if a distribution group with the same name as the OU exist
$OUs | %{$OU = $_.DistinguishedName;
    if(Get-ADGroup -Filter {SamAccountName -eq $_.Name -and GroupCategory -eq &quot;Distribution&quot;} | Where-Object{$_.DistinguishedName -like &quot;*$OU&quot;}){

        Write-Host &quot;Update users in distribution group $($_.Name).&quot;
        $ADGroup = Get-ADGroup -Filter {SamAccountName -eq $_.Name -and GroupCategory -eq &quot;Distribution&quot;}
        Get-ADGroupMember -Identity $ADGroup | %{Remove-ADGroupMember -Identity $ADGroup -Members $_ -Confirm:$false}
        Get-ADUser -Filter {EmailAddress -like &quot;*&quot;} -SearchBase $OU | where {$_.enabled -eq $true -and -not ($ExcludeUsers -contains $_.Name)} | where{$_ -ne $null} | %{Add-ADGroupMember -Identity $ADGroup -Members $_}

    }else{

        Write-Host &quot;Create distribution group $($_.Name).&quot;
        New-ADGroup -Name $_.Name -SamAccountName $_.Name -GroupCategory Distribution -GroupScope Universal -DisplayName $_.Name -Path $($_.DistinguishedName) -Description &quot;Distribution group for $($_.Name).&quot;
        $ADGroup = Get-ADGroup $_.Name
        Get-ADUser -Filter {EmailAddress -like &quot;*&quot;} -SearchBase $_.DistinguishedName | where {$_.enabled -eq $true -and -not ($ExcludeUsers -contains $_.Name)} | where{$_ -ne $null} | %{Add-ADGroupMember -Identity $ADGroup -Members $_}
    }
}

# custom configuration
$Configs | %{

    $ADGroup = Get-ADGroup -Identity $_.Name
    $Config = $_

    if($_.Options -match &quot;UpdateFromGroups&quot;){

        Write-Host &quot;Add users from $($Config.AddGroups) to $($ADGroup.Name).&quot;
        Get-ADGroupMember -Identity $ADGroup | %{Remove-ADGroupMember -Identity $ADGroup -Members $_ -Confirm:$false}
        $Config.AddGroups | %{Get-ADGroupMember -Identity $_ -Recursive | Get-ADUser | where {($_.enabled -eq $true) -and -not ($ExcludeUsers -contains $_.Name)}} | select -Unique | %{Add-ADGroupMember -Identity $ADGroup -Members $_}

    }

    if($_.Options -match &quot;RemoveGroups&quot;){

        Write-Host &quot;Remove users from $($Config.RemoveGroups) in $($ADGroup.Name).&quot;
        $ADGroupMembers = Get-ADGroupMember -Identity $ADGroup
        $Config.RemoveGroups | %{Get-ADGroupMember -Identity $_ -Recursive | Get-ADUser | where {($ADGroupMembers -match $_) -and ($_.enabled -eq $true) -and -not ($ExcludeUsers -contains $_.Name)}} | select -Unique  | %{Remove-ADGroupMember -Identity $ADGroup -Members $_ -Confirm:$false}

    }
}

if($error){
    Send-PPErrorReport -FileName &quot;activedirectory.mail.config.xml&quot; -ScriptName $MyInvocation.InvocationName
}
[/code]

Latest version of this script: <a href="https://gist.github.com/6352037">https://gist.github.com/6352037</a>