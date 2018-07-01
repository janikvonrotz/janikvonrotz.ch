---
id: 414
title: 'Office365 and ADFS: Activate licenses for users depending on AD group membership'
date: 2013-08-13T07:48:44+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=414
permalink: /2013/08/13/office365-and-adfs-activate-licenses-for-users-depending-on-ad-group-membership/
dsq_thread_id:
  - "1602703987"
image: /wp-content/uploads/2013/08/microsoft-office-365-e1394705447131.jpg
categories:
  - Active Directory
  - Office 365
  - PowerShell
tags:
  - activedirectory
  - groups
  - license
  - membership
  - office365
  - powershell
  - users
---
On Office365 the users have to be licensed in order to get access to the Office365 application. I've developed a PowerShell script which add a license depending on the group membership in the ActiveDirectory.
<!--more-->

[code lang="ps"]
&lt;#
$Metadata = @{
    Title = &quot;Set Office365 Licenses by ActiveDirectory Group Membership&quot;
    Filename = &quot;Set-O365UserLicensesByADGroup.ps1&quot;
    Description = @&quot;
Adding license to a Office365 user as long the user is in the correct ActiveDirectory group
or in the white list, the users is active, the user has a mailbox.
The script will remove inactive licenses or if necessary replace them.
&quot;@
    Tags = &quot;powershell, activedirectory, office365, user, license, activation&quot;
    Project = &quot;&quot;
    Author = &quot;Janik von Rotz&quot;
    AuthorContact = &quot;https://janikvonrotz.ch&quot;
    CreateDate = &quot;2013-08-13&quot;
    LastEditDate = &quot;2013-09-30&quot;
    Url = &quot;https://gist.github.com/janikvonrotz/6218401&quot;
    Version = &quot;3.0.0&quot;
    License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit https://creativecommons.org/licenses/by-sa/3.0/ch/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#&gt;

#--------------------------------------------------#
# settings
#--------------------------------------------------#

$UsageLocation = &quot;CH&quot;

$WhiteList = @{
    UserPrincipalName = &quot;admin@vbluzern.onmicrosoft.com&quot;
    License = &quot;vbluzern:STANDARDPACK&quot;
},
@{
    UserPrincipalName = &quot;bison.testoff368@vbl.ch&quot;
    License = &quot;vbluzern:STANDARDPACK&quot;
},
@{
    UserPrincipalName = &quot;bison.test07@vbl.ch&quot;
    License = &quot;vbluzern:STANDARDPACK&quot;
},
@{
    UserPrincipalName = &quot;bison.test@vbl.ch&quot;
    License = &quot;vbluzern:STANDARDPACK&quot;
},
@{
    UserPrincipalName = &quot;bison.testoff367@vbl.ch&quot;
    License = &quot;vbluzern:STANDARDPACK&quot;
}


$LicenseConfig = @{
    Name = &quot;SharePoint Online Plan 1&quot;
    License = &quot;vbluzern:SHAREPOINTSTANDARD&quot;
    ADGroupSID = &quot;S-1-5-21-1744926098-708661255-2033415169-37562&quot; # SPO_SharePointOnlinePlan1License
    Rule = &quot;&quot;
},
@{
    Name = &quot;Enterprise Plan 1&quot;
    License = &quot;vbluzern:STANDARDPACK&quot;
    ADGroupSID = &quot;S-1-5-21-1744926098-708661255-2033415169-36657&quot; # SPO_365E1License
    Rule = &quot;MailBoxExistOnline&quot;
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

Write-Host &quot;Get Office365 users&quot;
# connect to office365
Connect-MsolService -Credential $Credential
$MsolUsers = Get-MsolUser -All

Write-Host &quot;Get ExchangeOnline mailboxes&quot;
# import session
$s = New-PSSession -ConfigurationName Microsoft.Exchange `
    -ConnectionUri https://ps.outlook.com/powershell `
    -Credential $(Get-Credential -Credential $Credential) `
    -Authentication Basic `
    -AllowRedirection
$EOMailboxes = Invoke-Command -Session $s -ScriptBlock{Get-MailBox} | select UserPrincipalName | %{&quot;$($_.UserPrincipalName)&quot;}
# remove session&quot;
Remove-PSSession $s

# combine users and their license
$LicenseAndUser = ($LicenseConfig |
    %{$License = $_ ; Get-ADGroupMember $_.ADGroupSID -Recursive | Get-ADUser | where{$_.Enabled -eq $true} |
    %{$_ | select UserPrincipalName, @{Name = &quot;License&quot;; Expression = {$License.License}}, @{Name = &quot;Rule&quot;; Expression = {$License.Rule}}}
    }) + ($WhiteList | select  @{Name = &quot;UserPrincipalName&quot;; Expression = {$_.UserPrincipalName}},
    @{Name = &quot;License&quot;; Expression = {$_.License}},
    @{Name = &quot;Rule&quot;; Expression = {&quot;&quot;}})


foreach($User in $MsolUsers){

    # first check whitelist
    $Config = $LicenseAndUser | where{$_.UserPrincipalName -eq $User.UserPrincipalName}

    if(($Config) -and ((($Config.Rule -eq &quot;MailBoxExistOnline&quot;) -and ($EOMailboxes -contains $User.UserPrincipalName)) -or ($Config.Rule -eq &quot;&quot;))){

        if($User.IsLicensed -and ($User.Licenses.AccountSkuId -ne $Config.License)){

            Write-Host &quot;Replace Office365 license: $($User.Licenses.AccountSkuId) with: $($Config.License) for user: $($User.UserPrincipalName)&quot;
            $User.Licenses | %{Set-MsolUserLicense -UserPrincipalName $User.UserPrincipalName -RemoveLicenses $_.AccountSkuId}
            Set-MsolUserLicense -UserPrincipalName $User.UserPrincipalName -AddLicenses $Config.License

        }elseif($User.IsLicensed){

            Write-Host &quot;User: $($User.UserPrincipalName) is already licensed with: $($Config.License)&quot;

        }else{

            Write-Host &quot;Set Office365 license: $($Config.License) for user: $($User.UserPrincipalName)&quot;
            Set-MsolUser -UserPrincipalName $User.UserPrincipalName -UsageLocation $UsageLocation
            Set-MsolUserLicense -UserPrincipalName $User.UserPrincipalName -AddLicenses $Config.License

        }

    }else{

        if($User.IsLicensed){

            Write-Host &quot;Remove Office365 license: $($User.Licenses.AccountSkuId) from user: $($User.UserPrincipalName)&quot;
            $User.Licenses | %{Set-MsolUserLicense -UserPrincipalName $User.UserPrincipalName -RemoveLicenses $_.AccountSkuId}

        }else{

            Write-Host &quot;User: $($User.UserPrincipalName) is not allowed&quot;

        }
    }
}

if($error){

    Send-PPErrorReport -FileName &quot;DirSync.mail.config.xml&quot; -ScriptName $MyInvocation.InvocationName

}
[/code]

You'll get your <code>Office365AccountName</code> with this Office365 command <code>Get-MsolAccountSku</code>

Link to the newest version of this script: <a href="https://gist.github.com/janikvonrotz/6218401">https://gist.github.com/janikvonrotz/6218401</a>