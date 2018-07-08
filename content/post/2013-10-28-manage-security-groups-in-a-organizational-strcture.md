---
title: Manage Security Groups in a organizational Strcture
date: 2013-10-28T16:22:20+00:00
author: Janik von Rotz
slug: manage-security-groups-in-a-organizational-strcture
dsq_thread_id:
  - "1910803381"
image: /wp-content/uploads/2013/07/PowerShell.png
categories:
  - Active Directory
  - PowerShell
tags:
  - activedirectory
  - add
  - auto
  - group
  - management
  - organizational
  - powershell
  - remove
  - script
  - security
  - unit
  - update
---
As in on of my last <a title="Manage ActiveDirectory Distribution Groups" href="https://janikvonrotz.ch/2013/08/27/manage-activedirectory-distribution-groups/">post</a> I've showed you my approach to manage distribution groups in the hierarchical structure of an ActiveDirectory installation. In the mean time I've adapted a similiar approach for the security groups.

Here is an example of the structure:<!--more-->

<em>Existing OU structure and groups:</em>

<ul type="disc">
    <li>OU: Finance</li>
    <li>Group: F_accountant > Member: Hans Kleister</li>
</ul>

<ul type="disc">
<ul>
    <li>OU: Sale</li>
    <li>Group: F_Seller > Member: Lisa Meister</li>
</ul>
</ul>

<ul type="disc">
<ul>
    <li>OU: IT</li>
    <li>Group: F_System Engineer > Member: Johann Wagner</li>
</ul>
</ul>

And here after executing the management script:

<em>New Security groups :</em>

<ul type="disc">
    <li>OU: Finanze > Groups:
<ul type="disc">
    <li><strong>Finance Department</strong> > Member: F_accountant</li>
</ul>
<ul type="disc">
    <li><strong>Finance Departments</strong> > Member: Sale Department, IT Department
<ul type="circle">
    <li>OU: Sale > Groups:
<ul type="disc">
    <li><strong>Sale Department</strong> > Member: F_Seller</li>
</ul>
</li>
</ul>
<ul type="circle">
    <li>OU: IT > Groups:
<ul type="disc">
    <li><strong>IT Department</strong> > Member: F_System Engineer</li>
</ul>
</li>
</ul>
</li>
</ul>
</li>
</ul>

The rules

<ul>
    <li>Foreach OU create a department group: [Name + Defined Suffix]
<ul>
    <li>Add default members for the group: Filter by [Defined Prefix]</li>
    <li>Remove members which don't apply to the filter [Defined Prefix]</li>
</ul>
</li>
    <li>If OU contains OUs with department groups, create a departement<strong>s</strong> group
<ul>
    <li>Members are the child department groups.</li>
</ul>
</li>
</ul>

As always you can manage special settings with Exclude-Filters and simple Tasks.

And here's the script:

```powershell
<#
$Metadata = @{
    Title = "Update ActiveDirectory Security Groups"
    Filename = "Update-ADSecurityGroups.ps1"
    Description = ""
    Tags = "powershell, activedirectory, security, groups, update"
    Project = ""
    Author = "Janik von Rotz"
    AuthorContact = "https://janikvonrotz.ch"
    CreateDate = "2013-10-07"
    LastEditDate = "2013-10-24"
    Url = ""
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

$ExcludeADUsers = ""
$ExcludeADGroups = ""

$OUConfigs = @(
    @{
        OU = "OU=vblusers2,DC=vbl,DC=ch"

        GroupSuffix = " Abteilung"
        GroupMemberPrefix = "F_"

        ParentGroupSuffix = " Abteilungen"
        ParentGroupMemberSuffix = " Abteilung"

        ExcludeOUs = "Extern","ServiceAccounts","Services"

        ExcludeADUsers = "" + $ExcludeADUsers
        ExcludeADGroups = "F_Verwaltungsrat" + $ExcludeADGroups
    }
)

$Tasks = @(
    @{
        Name = "SP_Home#Read";
        Options = @("CleanGroup","UpdateFromGroups","ProcessUsers");
        AddGroups = @("Technik Abteilungen","Betrieb Abteilungen","Personal Abteilungen","Finanzen Abteilungen","Direktion Abteilungen","F_TermUser")
    },
    @{
        Name = "SP_Home#Edit";
        Options = @("CleanGroup","UpdateFromGroups","RemoveGroups","ProcessUsers");
        AddGroups = @("SP_Home#Read");
        RemoveGroups = @("SPO_365E1License","F_TermUser","F_Service Benutzer")
    }

)

$OUConfigs | %{
    $OUConfig = $_
    Get-ADOrganizationalUnit -Filter "*" -SearchBase $_.OU |
    where{$ThisOU = $_; -not ($OUConfig.ExcludeOUs | where{$ThisOU.DistinguishedName -match $_})} | %{

        $OUconfig.OU = $_

        $ParentGroupName = ($_.Name + $OUconfig.ParentGroupSuffix)
        $ParentGroupMembers = Get-ADOrganizationalUnit -Filter * -SearchBase $_.DistinguishedName | %{Get-ADGroup -SearchScope OneLevel -Filter * -SearchBase $_.DistinguishedName | where{$_.Name.EndsWith($OUconfig.ParentGroupMemberSuffix)}} | select -Unique
        $ParentGroup = Get-ADGroup -SearchScope OneLevel -Filter {SamAccountName -eq $ParentGroupName -and GroupCategory -eq "Security"}  -SearchBase $_.DistinguishedName

        $GroupName = ($_.Name + $OUconfig.GroupSuffix)
        $GroupMembers = Get-ADGroup -SearchScope OneLevel -Filter * -SearchBase $_.DistinguishedName | where{$_.Name.StartsWith($OUconfig.GroupMemberPrefix) -and ($OUconfig.ExcludeADGroups -notcontains $_.Name)}
        $Group = Get-ADGroup -SearchScope OneLevel -Filter{SamAccountName -eq $GroupName -and GroupCategory -eq "Security"} -SearchBase $_.DistinguishedName

        if($ParentGroupMembers -and $ParentGroup){

            "Update members in parent group: $($ParentGroup.Name)." | %{$Message += "`n" + $_; Write-Host $_}
            Get-ADGroupMember -Identity $ParentGroup | %{Remove-ADGroupMember -Identity $ParentGroup -Members $_ -Confirm:$false}
            $ParentGroupMembers | %{Add-ADGroupMember -Identity $ParentGroup -Members $_}

        }elseif($ParentGroupMembers -and $ParentGroupMembers.count -gt 1){

            "Add parent group: $ParentGroupName." | %{$Message += "`n" + $_; Write-Host $_}
            New-ADGroup -Name $ParentGroupName -SamAccountName $ParentGroupName -GroupCategory Security -GroupScope Global -DisplayName $ParentGroupName -Path $($OU.DistinguishedName) -Description "Department group for $($OU.Name)"
            $ParentGroupMembers | %{Add-ADGroupMember -Identity $ParentGroupName -Members $_}
        }

        if($Group -and $GroupMembers){

            #"Update members in group: $($Group.Name)." | %{$Message += "`n" + $_; Write-Host $_}
            $GroupMembersIS = Get-ADGroupMember -Identity $Group | %{"$($_.DistinguishedName)"}
            $GroupMemberTO = $GroupMembers | %{"$($_.DistinguishedName)"}

            Get-ADGroupMember -Identity $Group | where{(-not $_.Name.StartsWith($OUconfig.GroupMemberPrefix)) -or ($GroupMemberTO -notcontains $_.DistinguishedName)} | %{
                "Remove member: $($_.Name) from group: $($Group.Name)." | %{$Message += "`n" + $_; Write-Host $_}
                Remove-ADGroupMember -Identity $Group -Members $_ -Confirm:$false
            }

            $GroupMembers | where{($GroupMembersIS -notcontains $_.DistinguishedName)} | %{
                "Add member: $($_.Name) to group: $($Group.Name)." | %{$Message += "`n" + $_; Write-Host $_}
                Add-ADGroupMember -Identity $Group -Members $_
            }

        }elseif($GroupMembers){

            "Add group: $GroupName." | %{$Message += "`n" + $_; Write-Host $_}
            New-ADGroup -Name $GroupName -SamAccountName $GroupName -GroupCategory Security -GroupScope Global -DisplayName $GroupName -Path $($OU.DistinguishedName) -Description "Department group for $($OU.Name)"
            $GroupMembers | %{Add-ADGroupMember -Identity $GroupName -Members $_}
        }
    }
}

$Tasks | %{

    $ADGroup = Get-ADGroup -Identity $_.Name

    if($_.Options -match "CleanGroup"){

        "Remove members from: $($_.Name)." | %{$Message += "`n" + $_; Write-Host $_}
        Get-ADGroupMember -Identity $ADGroup | %{Remove-ADGroupMember -Identity $ADGroup -Members $_ -Confirm:$false}
    }

    if($_.Options -match "UpdateFromGroups"){

        if($_.Options -match "ProcessUsers"){

            "Add users from: $($_.AddGroups) to: $($_.Name)." | %{$Message += "`n" + $_; Write-Host $_}
            $_.AddGroups | %{Get-ADGroupMember $_ -Recursive | Get-ADUser | where {($_.Enabled -eq $true) -and ($ExcludeADUsers -notcontains $_.UserPrincipalName)}} | select -Unique | %{Add-ADGroupMember -Identity $ADGroup -Members $_}

        }else{

            "Add groups: $($_.AddGroups) to: $($_.Name)." | %{$Message += "`n" + $_; Write-Host $_}
            $_.AddGroups | %{Add-ADGroupMember -Identity $ADGroup -Members $_}
        }
    }

    if($_.Options -match "RemoveGroups"){

        if($_.Options -match "ProcessUsers"){

            "Remove users from: $($_.RemoveGroups) to: $($_.Name)." | %{$Message += "`n" + $_; Write-Host $_}
            $ADGroupMembers = Get-ADGroupMember -Identity $ADGroup
            $_.RemoveGroups | %{Get-ADGroupMember $_ -Recursive | Get-ADUser | where {($ADGroupMembers -match $_) -and ($_.Enabled -eq $true) -and ($ExcludeADUsers -notcontains $_.UserPrincipalName)}} | select -Unique |  %{Remove-ADGroupMember -Identity $ADGroup -Members $_  -Confirm:$false}

        }else{

            "Remove groups: $($_.RemoveGroups) to: $($_.Name)." | %{$Message += "`n" + $_; Write-Host $_}
            $_.RemoveGroups | %{Remove-ADGroupMember -Identity $ADGroup -Members $_ -Confirm:$false}
        }
    }
}

Write-PPEventLog $($MyInvocation.InvocationName + "`n`n" + $Message )
Send-PPErrorReport -FileName "activedirectory.mail.config.xml" -ScriptName $MyInvocation.InvocationName
Write-PPErrorEventLog -ScriptPath $MyInvocation.InvocationName -ClearErrorVariable
```

<ul>
    <li>Many functions are part of my project: <a href="https://github.com/janikvonrotz/Powershell-Profile">https://github.com/janikvonrotz/Powershell-Profile</a></li>
    <li>Get the latest version of this script here:<a href="https://gist.github.com/7137592" target="_blank"> https://gist.github.com/7137592</a></li>
</ul>