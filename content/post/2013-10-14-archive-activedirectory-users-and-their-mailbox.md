---
title: Archive ActiveDirectory Users and their Mailbox
date: 2013-10-14T17:04:19+00:00
author: Janik von Rotz
slug: archive-activedirectory-users-and-their-mailbox
dsq_thread_id:
  - "1856361269"
image: /wp-content/uploads/2013/08/exchange-2013-e1393417827333.jpg
categories:
  - Active Directory
  - Exchange
  - PowerShell
tags:
  - activedirectory
  - advanced
  - archive
  - configuration
  - cycle
  - exchange
  - life
  - mailbox
  - management
  - powershell
  - script
  - snippet
  - users
---
One of my company's requirements is the retention time of 10 years for user accounts and their mailbox data, I have to admit, this might not be common or even recommended.
However I have to deal with it.

One problem to face is the availabilty of user account names, by the number of about 500 employees there's a hight change that two or even more people are having the same name.

To clean up the available names in the system I've written a script that renames a users identity and the mailboxes address.
So let's see what this script does:

<!--more-->

<ol>
    <li>Filter all disabled users having an mailbox and who are visible in the exchange address book.</li>
    <li>Hide the mailbox address in all exchange addressbooks.</li>
    <li>Rename the ActiveDirectory user object by salting the name with the user's SID.</li>
    <li>Do the same for the mail addresses</li>
</ol>

```powershell
#--------------------------------------------------#
# settings
#--------------------------------------------------#
$ExchangeServer = "vblw2k8mail05"
$FilterRecipientTypeDetails = @("UserMailbox","RemoteUserMailbox")

#--------------------------------------------------#
# functions
#--------------------------------------------------#

function Rename-ADUserAndMailbox{

    param(
        [Parameter(Mandatory=$true)]
        $ADUser,

        [Parameter(Mandatory=$true)]
        $MailBox
    )

    $ArchivedIdentity = ($($ADUser.SID).tostring() -replace "-","").substring(20)

    if(-not (Get-ADUser -Filter{SamAccountName -eq $ArchivedIdentity} -ErrorAction SilentlyContinue)){

        $NewName = "$($ADUser.Name) $($ADUser.SID)"
        $NewUserPrincipalName =  "$($ADUser.UserPrincipalName.split('@')[0]) $($ADUser.SID)@$($ADUser.UserPrincipalName.split('@')[1])"
        $NewSamAccountName = ($($ADUser.SID).tostring() -replace "-","").substring(20)

        Write-Host "Rename Name $($ADUser.Name) to $NewName"
        Rename-ADObject $ADUser -NewName $NewName

        Write-Host "Rename UserPrincipalName $($ADUser.UserPrincipalName) to $NewUserPrincipalName"
        Get-ADUser $ADUser.SamAccountName | Set-ADUser -UserPrincipalName $NewUserPrincipalName -Description "archived"

        Write-Host "Rename SamAccountName $($ADUser.SamAccountName) to $NewSamAccountName"
        Get-ADUser $ADUser.SamAccountName | Set-ADUser -SamAccountName $NewSamAccountName

        $NewPrimarySmtpAddress = "$($ADUser.UserPrincipalName.split('@')[0])$($ADUser.SID)@$($ADUser.UserPrincipalName.split('@')[1])" -replace "-",""
        $OldPrimarySmtpAddress = $Mailbox.PrimarySmtpAddress

        if($Mailbox.psObject.TypeNames -contains "Deserialized.Microsoft.Exchange.Data.Directory.Management.RemoteMailbox"){

            $NewRemoteRoutingAddress = "$($Mailbox.RemoteRoutingAddress.split("@")[0])$($ADUser.SID)@$($Mailbox.RemoteRoutingAddress.split("@")[1])" -replace "-",""
            $OldRemoteRoutingAddress = $Mailbox.RemoteRoutingAddress

            Get-RemoteMailbox $ADuser.Name | %{

                Write-Host "Rename PrimarySmtpAddress for $($_.PrimarySmtpAddress) to $NewPrimarySmtpAddress"
                Set-RemoteMailbox $_.Alias -PrimarySmtpAddress $NewPrimarySmtpAddress;

                Write-Host "Rename RemoteRoutingAddress for $($_.RemoteRoutingAddress) to $NewRemoteRoutingAddress"
                Set-RemoteMailbox $_.Alias -RemoteRoutingAddress $NewRemoteRoutingAddress

                Write-Host "Remove default mail addresses $OldRemoteRoutingAddress, $PrimarySmtpAddress on $($_.Alias)"
                Set-RemoteMailbox $_.Alias -EmailAddresses @{remove = $OldRemoteRoutingAddress, $OldPrimarySmtpAddress}
            }

        }elseif($Mailbox.psObject.TypeNames -contains "Deserialized.Microsoft.Exchange.Data.Directory.Management.Mailbox"){

            Get-Mailbox $ADuser.Name | %{

                Write-Host "Rename PrimarySmtpAddress for $($_.PrimarySmtpAddress) to $NewPrimarySmtpAddress"
                Set-Mailbox $_.Alias -PrimarySmtpAddress $NewPrimarySmtpAddress

                Write-Host "Remove default mail addresses $PrimarySmtpAddress on $($Mailbox.Alias)"
                Set-Mailbox $_.Alias -EmailAddresses @{remove = $OldPrimarySmtpAddress}
            }
        }
    }
}

#--------------------------------------------------#
# modules
#--------------------------------------------------#
Import-Module ActiveDirectory

#--------------------------------------------------#
# main
#--------------------------------------------------#

# open remote connection
$PSSession = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri "https://$ExchangeServer/PowerShell/" -Authentication Kerberos

# import
Import-PSSession $PSSession -AllowClobber

$Mailboxes = Get-Mailbox
$RemoteMailboxes = Get-RemoteMailbox

# disable mailbox and remote mailbox
Get-ADUser -Filter{Enabled -eq $false} -Properties mail | where{$_.mail -ne $null} |
    %{$ADUser = $_; $Mailboxes | where{$_.Name -eq $ADuser.Name -and $_.HiddenFromAddressListsEnabled -eq $false -and $FilterRecipientTypeDetails -contains $_.RecipientTypeDetails}} |%{
        Write-host "Hide mailbox $($_.Name) from address lists.";
        Set-Mailbox $_.Name -HiddenFromAddressListsEnabled:$true;
        Rename-ADUserAndMailbox -ADUser $ADUser -MailBox $_
    }

# disable remote mailbox
Get-ADUser -Filter{Enabled -eq $false} -Properties mail | where{$_.mail -ne $null} |
    %{$ADUser = $_; $RemoteMailboxes | where{$_.Name -eq $ADuser.Name -and $_.HiddenFromAddressListsEnabled -eq $false -and $FilterRecipientTypeDetails -contains $_.RecipientTypeDetails}} | %{
        Write-host "Hide remotemailbox $($_.Name) from address lists.";
        Set-RemoteMailbox $_.Name -HiddenFromAddressListsEnabled:$true;
        Rename-ADUserAndMailbox -ADUser $ADUser -MailBox $_
}

# destroy pssession
Remove-PSSession $PSSession

if($error){
    Send-PPErrorReport -FileName "activedirectory.mail.config.xml" -ScriptName $MyInvocation.InvocationName
}

```

Latest version of this script: <a href="https://gist.github.com/6780143" target="_blank">https://gist.github.com/6780143</a>

As always I recommand you to install my project <a href="https://github.com/janikvonrotz/Powershell-Profile" target="_blank">PowerShell Profile</a> to run this script properly.