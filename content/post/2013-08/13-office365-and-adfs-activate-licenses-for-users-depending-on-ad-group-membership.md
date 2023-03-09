---
title: 'Office365 and ADFS: Activate licenses for users depending on AD group membership'
date: 2013-08-13T07:48:44+00:00
author: Janik von Rotz
slug: office365-and-adfs-activate-licenses-for-users-depending-on-ad-group-membership
images:
  - /wp-content/uploads/2013/08/microsoft-office-365-e1394705447131.jpg
categories:
  - Office 365
  - scripting
tags:
  - activedirectory
  - license
  - membership
  - office365
  - powershell
---
On Office365 the users have to be licensed in order to get access to the Office365 application. I've developed a PowerShell script which add a license depending on the group membership in the ActiveDirectory.
<!--more-->

```powershell
<#
$Metadata = @{
    Title = "Set Office365 Licenses by ActiveDirectory Group Membership"
    Filename = "Set-O365UserLicensesByADGroup.ps1"
    Description = @"
Adding license to a Office365 user as long the user is in the correct ActiveDirectory group
or in the white list, the users is active, the user has a mailbox.
The script will remove inactive licenses or if necessary replace them.
"@
    Tags = "powershell, activedirectory, office365, user, license, activation"
    Project = ""
    Author = "Janik von Rotz"
    AuthorContact = "https://janikvonrotz.ch"
    CreateDate = "2013-08-13"
    LastEditDate = "2013-09-30"
    Url = "https://gist.github.com/janikvonrotz/6218401"
    Version = "3.0.0"
    License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit https://creativecommons.org/licenses/by-sa/3.0/ch/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#>

#--------------------------------------------------#
# settings
#--------------------------------------------------#

$UsageLocation = "CH"

$WhiteList = @{
    UserPrincipalName = "admin@vbluzern.onmicrosoft.com"
    License = "vbluzern:STANDARDPACK"
},
@{
    UserPrincipalName = "bison.testoff368@vbl.ch"
    License = "vbluzern:STANDARDPACK"
},
@{
    UserPrincipalName = "bison.test07@vbl.ch"
    License = "vbluzern:STANDARDPACK"
},
@{
    UserPrincipalName = "bison.test@vbl.ch"
    License = "vbluzern:STANDARDPACK"
},
@{
    UserPrincipalName = "bison.testoff367@vbl.ch"
    License = "vbluzern:STANDARDPACK"
}


$LicenseConfig = @{
    Name = "SharePoint Online Plan 1"
    License = "vbluzern:SHAREPOINTSTANDARD"
    ADGroupSID = "S-1-5-21-1744926098-708661255-2033415169-37562" # SPO_SharePointOnlinePlan1License
    Rule = ""
},
@{
    Name = "Enterprise Plan 1"
    License = "vbluzern:STANDARDPACK"
    ADGroupSID = "S-1-5-21-1744926098-708661255-2033415169-36657" # SPO_365E1License
    Rule = "MailBoxExistOnline"
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

Write-Host "Get Office365 users"
# connect to office365
Connect-MsolService -Credential $Credential
$MsolUsers = Get-MsolUser -All

Write-Host "Get ExchangeOnline mailboxes"
# import session
$s = New-PSSession -ConfigurationName Microsoft.Exchange `
    -ConnectionUri https://ps.outlook.com/powershell `
    -Credential $(Get-Credential -Credential $Credential) `
    -Authentication Basic `
    -AllowRedirection
$EOMailboxes = Invoke-Command -Session $s -ScriptBlock{Get-MailBox} | select UserPrincipalName | %{"$($_.UserPrincipalName)"}
# remove session"
Remove-PSSession $s

# combine users and their license
$LicenseAndUser = ($LicenseConfig |
    %{$License = $_ ; Get-ADGroupMember $_.ADGroupSID -Recursive | Get-ADUser | where{$_.Enabled -eq $true} |
    %{$_ | select UserPrincipalName, @{Name = "License"; Expression = {$License.License}}, @{Name = "Rule"; Expression = {$License.Rule}}}
    }) + ($WhiteList | select  @{Name = "UserPrincipalName"; Expression = {$_.UserPrincipalName}},
    @{Name = "License"; Expression = {$_.License}},
    @{Name = "Rule"; Expression = {""}})


foreach($User in $MsolUsers){

    # first check whitelist
    $Config = $LicenseAndUser | where{$_.UserPrincipalName -eq $User.UserPrincipalName}

    if(($Config) -and ((($Config.Rule -eq "MailBoxExistOnline") -and ($EOMailboxes -contains $User.UserPrincipalName)) -or ($Config.Rule -eq ""))){

        if($User.IsLicensed -and ($User.Licenses.AccountSkuId -ne $Config.License)){

            Write-Host "Replace Office365 license: $($User.Licenses.AccountSkuId) with: $($Config.License) for user: $($User.UserPrincipalName)"
            $User.Licenses | %{Set-MsolUserLicense -UserPrincipalName $User.UserPrincipalName -RemoveLicenses $_.AccountSkuId}
            Set-MsolUserLicense -UserPrincipalName $User.UserPrincipalName -AddLicenses $Config.License

        }elseif($User.IsLicensed){

            Write-Host "User: $($User.UserPrincipalName) is already licensed with: $($Config.License)"

        }else{

            Write-Host "Set Office365 license: $($Config.License) for user: $($User.UserPrincipalName)"
            Set-MsolUser -UserPrincipalName $User.UserPrincipalName -UsageLocation $UsageLocation
            Set-MsolUserLicense -UserPrincipalName $User.UserPrincipalName -AddLicenses $Config.License

        }

    }else{

        if($User.IsLicensed){

            Write-Host "Remove Office365 license: $($User.Licenses.AccountSkuId) from user: $($User.UserPrincipalName)"
            $User.Licenses | %{Set-MsolUserLicense -UserPrincipalName $User.UserPrincipalName -RemoveLicenses $_.AccountSkuId}

        }else{

            Write-Host "User: $($User.UserPrincipalName) is not allowed"

        }
    }
}

if($error){

    Send-PPErrorReport -FileName "DirSync.mail.config.xml" -ScriptName $MyInvocation.InvocationName

}
```

You'll get your `Office365AccountName` with this Office365 command `Get-MsolAccountSku`

Link to the newest version of this script: <a href="https://gist.github.com/janikvonrotz/6218401">https://gist.github.com/janikvonrotz/6218401</a>