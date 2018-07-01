---
id: 653
title: Manage Security Groups in a organizational Strcture
date: 2013-10-28T16:22:20+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=653
permalink: /2013/10/28/manage-security-groups-in-a-organizational-strcture/
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
    <li>Group: F_accountant &gt; Member: Hans Kleister</li>
</ul>

<ul type="disc">
<ul>
    <li>OU: Sale</li>
    <li>Group: F_Seller &gt; Member: Lisa Meister</li>
</ul>
</ul>

<ul type="disc">
<ul>
    <li>OU: IT</li>
    <li>Group: F_System Engineer &gt; Member: Johann Wagner</li>
</ul>
</ul>

And here after executing the management script:

<em>New Security groups :</em>

<ul type="disc">
    <li>OU: Finanze &gt; Groups:
<ul type="disc">
    <li><strong>Finance Department</strong> &gt; Member: F_accountant</li>
</ul>
<ul type="disc">
    <li><strong>Finance Departments</strong> &gt; Member: Sale Department, IT Department
<ul type="circle">
    <li>OU: Sale &gt; Groups:
<ul type="disc">
    <li><strong>Sale Department</strong> &gt; Member: F_Seller</li>
</ul>
</li>
</ul>
<ul type="circle">
    <li>OU: IT &gt; Groups:
<ul type="disc">
    <li><strong>IT Department</strong> &gt; Member: F_System Engineer</li>
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

[code lang="ps"]
&lt;#
$Metadata = @{
    Title = &quot;Update ActiveDirectory Security Groups&quot;
    Filename = &quot;Update-ADSecurityGroups.ps1&quot;
    Description = &quot;&quot;
    Tags = &quot;powershell, activedirectory, security, groups, update&quot;
    Project = &quot;&quot;
    Author = &quot;Janik von Rotz&quot;
    AuthorContact = &quot;https://janikvonrotz.ch&quot;
    CreateDate = &quot;2013-10-07&quot;
    LastEditDate = &quot;2013-10-24&quot;
    Url = &quot;&quot;
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

$ExcludeADUsers = &quot;&quot;
$ExcludeADGroups = &quot;&quot;

$OUConfigs = @(
    @{
        OU = &quot;OU=vblusers2,DC=vbl,DC=ch&quot;

        GroupSuffix = &quot; Abteilung&quot;
        GroupMemberPrefix = &quot;F_&quot;

        ParentGroupSuffix = &quot; Abteilungen&quot;
        ParentGroupMemberSuffix = &quot; Abteilung&quot;

        ExcludeOUs = &quot;Extern&quot;,&quot;ServiceAccounts&quot;,&quot;Services&quot;

        ExcludeADUsers = &quot;&quot; + $ExcludeADUsers
        ExcludeADGroups = &quot;F_Verwaltungsrat&quot; + $ExcludeADGroups
    }
)

$Tasks = @(
    @{
        Name = &quot;SP_Home#Read&quot;;
        Options = @(&quot;CleanGroup&quot;,&quot;UpdateFromGroups&quot;,&quot;ProcessUsers&quot;);
        AddGroups = @(&quot;Technik Abteilungen&quot;,&quot;Betrieb Abteilungen&quot;,&quot;Personal Abteilungen&quot;,&quot;Finanzen Abteilungen&quot;,&quot;Direktion Abteilungen&quot;,&quot;F_TermUser&quot;)
    },
    @{
        Name = &quot;SP_Home#Edit&quot;;
        Options = @(&quot;CleanGroup&quot;,&quot;UpdateFromGroups&quot;,&quot;RemoveGroups&quot;,&quot;ProcessUsers&quot;);
        AddGroups = @(&quot;SP_Home#Read&quot;);
        RemoveGroups = @(&quot;SPO_365E1License&quot;,&quot;F_TermUser&quot;,&quot;F_Service Benutzer&quot;)
    }

)

$OUConfigs | %{
    $OUConfig = $_
    Get-ADOrganizationalUnit -Filter &quot;*&quot; -SearchBase $_.OU |
    where{$ThisOU = $_; -not ($OUConfig.ExcludeOUs | where{$ThisOU.DistinguishedName -match $_})} | %{

        $OUconfig.OU = $_

        $ParentGroupName = ($_.Name + $OUconfig.ParentGroupSuffix)
        $ParentGroupMembers = Get-ADOrganizationalUnit -Filter * -SearchBase $_.DistinguishedName | %{Get-ADGroup -SearchScope OneLevel -Filter * -SearchBase $_.DistinguishedName | where{$_.Name.EndsWith($OUconfig.ParentGroupMemberSuffix)}} | select -Unique
        $ParentGroup = Get-ADGroup -SearchScope OneLevel -Filter {SamAccountName -eq $ParentGroupName -and GroupCategory -eq &quot;Security&quot;}  -SearchBase $_.DistinguishedName

        $GroupName = ($_.Name + $OUconfig.GroupSuffix)
        $GroupMembers = Get-ADGroup -SearchScope OneLevel -Filter * -SearchBase $_.DistinguishedName | where{$_.Name.StartsWith($OUconfig.GroupMemberPrefix) -and ($OUconfig.ExcludeADGroups -notcontains $_.Name)}
        $Group = Get-ADGroup -SearchScope OneLevel -Filter{SamAccountName -eq $GroupName -and GroupCategory -eq &quot;Security&quot;} -SearchBase $_.DistinguishedName

        if($ParentGroupMembers -and $ParentGroup){

            &quot;Update members in parent group: $($ParentGroup.Name).&quot; | %{$Message += &quot;`n&quot; + $_; Write-Host $_}
            Get-ADGroupMember -Identity $ParentGroup | %{Remove-ADGroupMember -Identity $ParentGroup -Members $_ -Confirm:$false}
            $ParentGroupMembers | %{Add-ADGroupMember -Identity $ParentGroup -Members $_}

        }elseif($ParentGroupMembers -and $ParentGroupMembers.count -gt 1){

            &quot;Add parent group: $ParentGroupName.&quot; | %{$Message += &quot;`n&quot; + $_; Write-Host $_}
            New-ADGroup -Name $ParentGroupName -SamAccountName $ParentGroupName -GroupCategory Security -GroupScope Global -DisplayName $ParentGroupName -Path $($OU.DistinguishedName) -Description &quot;Department group for $($OU.Name)&quot;
            $ParentGroupMembers | %{Add-ADGroupMember -Identity $ParentGroupName -Members $_}
        }

        if($Group -and $GroupMembers){

            #&quot;Update members in group: $($Group.Name).&quot; | %{$Message += &quot;`n&quot; + $_; Write-Host $_}
            $GroupMembersIS = Get-ADGroupMember -Identity $Group | %{&quot;$($_.DistinguishedName)&quot;}
            $GroupMemberTO = $GroupMembers | %{&quot;$($_.DistinguishedName)&quot;}

            Get-ADGroupMember -Identity $Group | where{(-not $_.Name.StartsWith($OUconfig.GroupMemberPrefix)) -or ($GroupMemberTO -notcontains $_.DistinguishedName)} | %{
                &quot;Remove member: $($_.Name) from group: $($Group.Name).&quot; | %{$Message += &quot;`n&quot; + $_; Write-Host $_}
                Remove-ADGroupMember -Identity $Group -Members $_ -Confirm:$false
            }

            $GroupMembers | where{($GroupMembersIS -notcontains $_.DistinguishedName)} | %{
                &quot;Add member: $($_.Name) to group: $($Group.Name).&quot; | %{$Message += &quot;`n&quot; + $_; Write-Host $_}
                Add-ADGroupMember -Identity $Group -Members $_
            }

        }elseif($GroupMembers){

            &quot;Add group: $GroupName.&quot; | %{$Message += &quot;`n&quot; + $_; Write-Host $_}
            New-ADGroup -Name $GroupName -SamAccountName $GroupName -GroupCategory Security -GroupScope Global -DisplayName $GroupName -Path $($OU.DistinguishedName) -Description &quot;Department group for $($OU.Name)&quot;
            $GroupMembers | %{Add-ADGroupMember -Identity $GroupName -Members $_}
        }
    }
}

$Tasks | %{

    $ADGroup = Get-ADGroup -Identity $_.Name

    if($_.Options -match &quot;CleanGroup&quot;){

        &quot;Remove members from: $($_.Name).&quot; | %{$Message += &quot;`n&quot; + $_; Write-Host $_}
        Get-ADGroupMember -Identity $ADGroup | %{Remove-ADGroupMember -Identity $ADGroup -Members $_ -Confirm:$false}
    }

    if($_.Options -match &quot;UpdateFromGroups&quot;){

        if($_.Options -match &quot;ProcessUsers&quot;){

            &quot;Add users from: $($_.AddGroups) to: $($_.Name).&quot; | %{$Message += &quot;`n&quot; + $_; Write-Host $_}
            $_.AddGroups | %{Get-ADGroupMember $_ -Recursive | Get-ADUser | where {($_.Enabled -eq $true) -and ($ExcludeADUsers -notcontains $_.UserPrincipalName)}} | select -Unique | %{Add-ADGroupMember -Identity $ADGroup -Members $_}

        }else{

            &quot;Add groups: $($_.AddGroups) to: $($_.Name).&quot; | %{$Message += &quot;`n&quot; + $_; Write-Host $_}
            $_.AddGroups | %{Add-ADGroupMember -Identity $ADGroup -Members $_}
        }
    }

    if($_.Options -match &quot;RemoveGroups&quot;){

        if($_.Options -match &quot;ProcessUsers&quot;){

            &quot;Remove users from: $($_.RemoveGroups) to: $($_.Name).&quot; | %{$Message += &quot;`n&quot; + $_; Write-Host $_}
            $ADGroupMembers = Get-ADGroupMember -Identity $ADGroup
            $_.RemoveGroups | %{Get-ADGroupMember $_ -Recursive | Get-ADUser | where {($ADGroupMembers -match $_) -and ($_.Enabled -eq $true) -and ($ExcludeADUsers -notcontains $_.UserPrincipalName)}} | select -Unique |  %{Remove-ADGroupMember -Identity $ADGroup -Members $_  -Confirm:$false}

        }else{

            &quot;Remove groups: $($_.RemoveGroups) to: $($_.Name).&quot; | %{$Message += &quot;`n&quot; + $_; Write-Host $_}
            $_.RemoveGroups | %{Remove-ADGroupMember -Identity $ADGroup -Members $_ -Confirm:$false}
        }
    }
}

Write-PPEventLog $($MyInvocation.InvocationName + &quot;`n`n&quot; + $Message )
Send-PPErrorReport -FileName &quot;activedirectory.mail.config.xml&quot; -ScriptName $MyInvocation.InvocationName
Write-PPErrorEventLog -ScriptPath $MyInvocation.InvocationName -ClearErrorVariable
[/code]

<ul>
    <li>Many functions are part of my project: <a href="https://github.com/janikvonrotz/Powershell-Profile">https://github.com/janikvonrotz/Powershell-Profile</a></li>
    <li>Get the latest version of this script here:<a href="https://gist.github.com/7137592" target="_blank"> https://gist.github.com/7137592</a></li>
</ul>