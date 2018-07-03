---
id: 453
title: Create a Office365 user license report
date: 2013-08-23T12:14:46+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=453
permalink: /2013/08/23/create-a-office365-user-license-report/
dsq_thread_id:
  - "1633015026"
image: /wp-content/uploads/2013/08/microsoft-office-365-e1394705447131.jpg
categories:
  - Office 365
  - PowerShell
tags:
  - cost
  - custom
  - gist
  - license
  - office365
  - report
  - script
  - snippet
---
With over 350 users in the Office365 cloud as in my case it's difficult being aware of which licenses I really need.

To help my out I've made an ActiveDirectory group which holds the allowed Office365 users. And with this PowerShell script I look up every Office365 user and his licenses and check if this users is allowed to use Office365.

<!--more-->

```ps
#--------------------------------------------------#
# modules
#--------------------------------------------------#

Import-Module MSOnline
Import-Module MSOnlineExtended
Import-Module ActiveDirectory

#--------------------------------------------------#
# main
#--------------------------------------------------#

# declaration
$Licenses = @()

# import credentials
$Credential = Import-PSCredential $(Get-ChildItem -Path $PSconfigs.Path -Filter "Office365.credentials.config.xml" -Recurse).FullName

# SID is SPO_365Users
Write-Host "Get allowed ActiveDirectory users"
$AllowADUsers = Get-ADGroupMember "S-1-5-21-1744926098-708661255-2033415169-36655" -Recursive | Get-ADUser | where {$_.enabled -eq $true} | select userprincipalname # SPO_365Users

# connect to office365
Connect-MsolService -Credential $Credential

Write-Host "Get Office365 users"
$MSOUsers = Get-MsolUser -All

Write-Host "Get ExchangeOnline mailboxes"
# import session
$s = New-PSSession -ConfigurationName Microsoft.Exchange `
    -ConnectionUri https://ps.outlook.com/powershell `
    -Credential $(Get-Credential -Credential $Credential) `
    -Authentication Basic `
    -AllowRedirection
Import-PSSession $s

# get upn of every mailbox
$MSOMailboxes = Get-Mailbox | select UserPrincipalName

$MSOUsers | foreach{

    Write-Progress -Activity "Report licenses" -status $_.UserPrincipalName -percentComplete ([int]([array]::IndexOf(([array]$MSOUsers), $_)/([array]$MSOUsers).count*100))

    $UserPrincipleName = $_.UserPrincipalName

    $License = $_ | Select-Object -Property UserPrincipalName,
    @{ Name = "Package";
        Expression = {
            $_.Licenses | ForEach-Object{$_.AccountSkuId}
        }
    },
    @{ Name = "Licenses";
        Expression = {
            $_.Licenses | ForEach-Object{
                $_.ServiceStatus | Where-Object{$_.ProvisioningStatus -ne "Disabled"}
            } | ForEach-Object{
                $_.ServicePlan.ServiceName
            }
        }
    },
    @{ Name = "IsAllowedOffice365User";
        Expression = {
            if($AllowADUsers -match $UserPrincipleName){$true}else{$false}
        }
    },
    @{ Name = "HasMailboxInCloud";
        Expression = {
            if($MSOMailboxes -match $UserPrincipleName){$true}else{$false}
        }
    }

    $Licenses += $License
}

$Licenses | Out-GridView

if($error){

    Send-PPErrorReport -FileName "DirSync.mail.config.xml" -ScriptName $MyInvocation.InvocationName

}
```

Latest version of this snippet: <a href="https://gist.github.com/6317367" target="_blank">https://gist.github.com/6317367</a>

<h1>Requirements</h1>

<ul>
    <li>The function `Import-PSCredential` is part of my PowerShell Profile project</li>
    <li>Further you have to install the ActiveDirectory and Office365 PowerShell CMDlets.</li>
</ul>