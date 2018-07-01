---
id: 540
title: Manage access rights to the Office365 portal
date: 2013-09-30T14:24:08+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=540
permalink: /2013/09/30/manage-the-access-rights-to-the-office365-portal/
dsq_thread_id:
  - "1811070729"
image: /wp-content/uploads/2013/09/office-logo_v3.jpg
categories:
  - Active Directory
  - Office 365
  - PowerShell
tags:
  - access
  - manage
  - office365
  - portal
  - rights
  - user
---
In addition to my last script showing how to manage the user licenses in Office365 I've written a new script for assign, remove or replace the access rights in the office365 portal.

The script has the same structure as the license management script, feel free as always to copy and alter this script or asking me questions about it.

<!--more-->

[code lang="ps"]
&lt;#
$Metadata = @{
    Title = &quot;Set Office365 User Rights&quot;
    Filename = &quot;Set-O365UserRights.ps1&quot;
    Description = @&quot;
Manage Office365 portal access rights with ActiveDirectory groups.
Assign Administration roles to the members of specified AD groups or by a users userprincipalname.
&quot;@
    Tags = &quot;powershell, activedirectory, office365, user, rights&quot;
    Project = &quot;&quot;
    Author = &quot;Janik von Rotz&quot;
    AuthorContact = &quot;https://janikvonrotz.ch&quot;
    CreateDate = &quot;2013-08-13&quot;
    LastEditDate = &quot;2013-09-26&quot;
    Url = &quot;https://gist.github.com/janikvonrotz/6218401&quot;
    Version = &quot;3.0.0&quot;
    License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit https://creativecommons.org/licenses/by-sa/3.0/ch/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#&gt;

try{

    #--------------------------------------------------#
    # settings
    #--------------------------------------------------#

    $MsolRoleConfig = @{
        ADGroup = &quot;S-1-5-21-1744926098-708661255-2033415169-37011&quot; # O365F_Billing Administrator
        MsolRoleName = &quot;Billing Administrator&quot; # Get-MsolRole
    },
    @{
        User = &quot;admin@vbluzern.onmicrosoft.com&quot; # O365F_Billing Administrator
        MsolRoleName = &quot;Company Administrator&quot; # Get-MsolRole
    }

    #--------------------------------------------------#
    # modules
    #--------------------------------------------------#
    Import-Module MSOnline
    Import-Module MSOnlineExtended
    Import-Module ActiveDirectory

    #--------------------------------------------------#
    # main
    #--------------------------------------------------#

    # import credentials
    $Credential = Import-PSCredential $(Get-ChildItem -Path $PSconfigs.Path -Filter &quot;Office365.credentials.config.xml&quot; -Recurse).FullName

    # connect to office365
    Connect-MsolService -Credential $Credential

    $UserAndMsolRole = @(
        ($MsolRoleConfig | where{$_.ADGroup -ne $null} | %{$MsolRole = $_.MsolRoleName; $MsolRole = (Get-MsolRole | where{$_.Name -eq $MsolRole}); Get-ADGroupMember $_.ADGroup -Recursive |Get-ADUser | select UserPrincipalName, @{Name = &quot;MsolRole&quot;; Expression={$MsolRole}}}),
        ($MsolRoleConfig | where{$_.User -ne $null}| %{$MsolRole = $_.MsolRoleName; $_ | select @{L = &quot;UserPrincipalName&quot;; E = {$_.User}},@{L = &quot;MsolRole&quot;; E = {Get-MsolRole | where{$_.Name -eq $MsolRole}}}})
    )

    $MsolRoleMembers = Get-MsolRole | %{$MsolRole = $_; Get-MsolRoleMember -RoleObjectId $_.ObjectID -MemberObjectTypes User | where{$_.isLicensed} | select @{L = &quot;UserPrincipalName&quot;; E = {$_.EmailAddress}},@{L = &quot;MsolRole&quot;; E = {$MsolRole}}}

    (Get-MsolUser -All) | %{

        $MsolUser = $_
        $AlreadyAssigned = $MsolRoleMembers | where{$_.UserPrincipalName -eq $MsolUser.UserPrincipalName}
        $ToAssign = $UserAndMsolRole | where{$_.UserPrincipalName -eq $MsolUser.UserPrincipalName}

        if($AlreadyAssigned){

            if(($ToAssign) -and ($AlreadyAssigned.MsolRole.ObjectId -ne $ToAssign.MsolRole.ObjectId)){

                Write-Host &quot;Replace role: $($AlreadyAssigned.MsolRole.Name) with: $($ToAssign.MsolRole.Name) for: $($MsolUser.UserPrincipalName).&quot;
                Remove-MsolRoleMember -RoleMemberEmailAddress $MsolUser.UserPrincipalName -RoleMemberType User -RoleName $AlreadyAssigned.MsolRole.Name
                Add-MsolRoleMember -RoleMemberEmailAddress $MsolUser.UserPrincipalName -RoleMemberType User -RoleName $ToAssign.MsolRole.Name

            }elseif($ToAssign -eq $null){

                Write-Host &quot;Remove role: $($AlreadyAssigned.MsolRole.Name) for: $($MsolUser.UserPrincipalName).&quot;
                Remove-MsolRoleMember -RoleMemberEmailAddress $MsolUser.UserPrincipalName -RoleMemberType User -RoleName $AlreadyAssigned.MsolRole.Name

            }else{

                Write-Host &quot;Role: $($AlreadyAssigned.MsolRole.Name) for: $($MsolUser.UserPrincipalName) is already assigned.&quot;

            }
        }elseif($ToAssign -and $MsolUser.IsLicensed){

            Write-Host &quot;Assign role: $($ToAssign.MsolRole.Name) for: $($MsolUser.UserPrincipalName).&quot;
            Add-MsolRoleMember -RoleMemberEmailAddress $MsolUser.UserPrincipalName -RoleMemberType User -RoleName $ToAssign.MsolRole.Name

        }elseif($ToAssign){

            throw &quot;Not possible to assign role: $($ToAssign.MsolRole.Name) user: $($MsolUser.UserPrincipalName) has to be licensed.&quot;

        }
    }

}catch{

    Send-PPErrorReport -FileName &quot;DirSync.mail.config.xml&quot; -ScriptName $MyInvocation.InvocationName

}[/code]

<a href="https://gist.github.com/janikvonrotz/6763616">https://gist.github.com/janikvonrotz/6763616</a>

<h1>Requirements</h1>

<ul>
    <li><a href="https://github.com/janikvonrotz/Powershell-Profile">Powershell-Profile</a></li>
    <li>Office365 with ADFS and DirSync</li>
</ul>