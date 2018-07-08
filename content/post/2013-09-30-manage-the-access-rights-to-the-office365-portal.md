---
title: Manage access rights to the Office365 portal
date: 2013-09-30T14:24:08+00:00
author: Janik von Rotz
slug: manage-the-access-rights-to-the-office365-portal
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

```powershell
<#
$Metadata = @{
    Title = "Set Office365 User Rights"
    Filename = "Set-O365UserRights.ps1"
    Description = @"
Manage Office365 portal access rights with ActiveDirectory groups.
Assign Administration roles to the members of specified AD groups or by a users userprincipalname.
"@
    Tags = "powershell, activedirectory, office365, user, rights"
    Project = ""
    Author = "Janik von Rotz"
    AuthorContact = "https://janikvonrotz.ch"
    CreateDate = "2013-08-13"
    LastEditDate = "2013-09-26"
    Url = "https://gist.github.com/janikvonrotz/6218401"
    Version = "3.0.0"
    License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit https://creativecommons.org/licenses/by-sa/3.0/ch/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#>

try{

    #--------------------------------------------------#
    # settings
    #--------------------------------------------------#

    $MsolRoleConfig = @{
        ADGroup = "S-1-5-21-1744926098-708661255-2033415169-37011" # O365F_Billing Administrator
        MsolRoleName = "Billing Administrator" # Get-MsolRole
    },
    @{
        User = "admin@vbluzern.onmicrosoft.com" # O365F_Billing Administrator
        MsolRoleName = "Company Administrator" # Get-MsolRole
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
    $Credential = Import-PSCredential $(Get-ChildItem -Path $PSconfigs.Path -Filter "Office365.credentials.config.xml" -Recurse).FullName

    # connect to office365
    Connect-MsolService -Credential $Credential

    $UserAndMsolRole = @(
        ($MsolRoleConfig | where{$_.ADGroup -ne $null} | %{$MsolRole = $_.MsolRoleName; $MsolRole = (Get-MsolRole | where{$_.Name -eq $MsolRole}); Get-ADGroupMember $_.ADGroup -Recursive |Get-ADUser | select UserPrincipalName, @{Name = "MsolRole"; Expression={$MsolRole}}}),
        ($MsolRoleConfig | where{$_.User -ne $null}| %{$MsolRole = $_.MsolRoleName; $_ | select @{L = "UserPrincipalName"; E = {$_.User}},@{L = "MsolRole"; E = {Get-MsolRole | where{$_.Name -eq $MsolRole}}}})
    )

    $MsolRoleMembers = Get-MsolRole | %{$MsolRole = $_; Get-MsolRoleMember -RoleObjectId $_.ObjectID -MemberObjectTypes User | where{$_.isLicensed} | select @{L = "UserPrincipalName"; E = {$_.EmailAddress}},@{L = "MsolRole"; E = {$MsolRole}}}

    (Get-MsolUser -All) | %{

        $MsolUser = $_
        $AlreadyAssigned = $MsolRoleMembers | where{$_.UserPrincipalName -eq $MsolUser.UserPrincipalName}
        $ToAssign = $UserAndMsolRole | where{$_.UserPrincipalName -eq $MsolUser.UserPrincipalName}

        if($AlreadyAssigned){

            if(($ToAssign) -and ($AlreadyAssigned.MsolRole.ObjectId -ne $ToAssign.MsolRole.ObjectId)){

                Write-Host "Replace role: $($AlreadyAssigned.MsolRole.Name) with: $($ToAssign.MsolRole.Name) for: $($MsolUser.UserPrincipalName)."
                Remove-MsolRoleMember -RoleMemberEmailAddress $MsolUser.UserPrincipalName -RoleMemberType User -RoleName $AlreadyAssigned.MsolRole.Name
                Add-MsolRoleMember -RoleMemberEmailAddress $MsolUser.UserPrincipalName -RoleMemberType User -RoleName $ToAssign.MsolRole.Name

            }elseif($ToAssign -eq $null){

                Write-Host "Remove role: $($AlreadyAssigned.MsolRole.Name) for: $($MsolUser.UserPrincipalName)."
                Remove-MsolRoleMember -RoleMemberEmailAddress $MsolUser.UserPrincipalName -RoleMemberType User -RoleName $AlreadyAssigned.MsolRole.Name

            }else{

                Write-Host "Role: $($AlreadyAssigned.MsolRole.Name) for: $($MsolUser.UserPrincipalName) is already assigned."

            }
        }elseif($ToAssign -and $MsolUser.IsLicensed){

            Write-Host "Assign role: $($ToAssign.MsolRole.Name) for: $($MsolUser.UserPrincipalName)."
            Add-MsolRoleMember -RoleMemberEmailAddress $MsolUser.UserPrincipalName -RoleMemberType User -RoleName $ToAssign.MsolRole.Name

        }elseif($ToAssign){

            throw "Not possible to assign role: $($ToAssign.MsolRole.Name) user: $($MsolUser.UserPrincipalName) has to be licensed."

        }
    }

}catch{

    Send-PPErrorReport -FileName "DirSync.mail.config.xml" -ScriptName $MyInvocation.InvocationName

}```

[https://gist.github.com/janikvonrotz/6763616](https://gist.github.com/janikvonrotz/6763616)

<h1>Requirements</h1>

<ul>
    <li><a href="https://github.com/janikvonrotz/Powershell-Profile">Powershell-Profile</a></li>
    <li>Office365 with ADFS and DirSync</li>
</ul>