---
id: 620
title: Archive ActiveDirectory Users and their Mailbox
date: 2013-10-14T17:04:19+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=620
permalink: /2013/10/14/archive-activedirectory-users-and-their-mailbox/
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

[code lang="ps"]
#--------------------------------------------------#
# settings
#--------------------------------------------------#
$ExchangeServer = &quot;vblw2k8mail05&quot;
$FilterRecipientTypeDetails = @(&quot;UserMailbox&quot;,&quot;RemoteUserMailbox&quot;)

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

    $ArchivedIdentity = ($($ADUser.SID).tostring() -replace &quot;-&quot;,&quot;&quot;).substring(20)

    if(-not (Get-ADUser -Filter{SamAccountName -eq $ArchivedIdentity} -ErrorAction SilentlyContinue)){

        $NewName = &quot;$($ADUser.Name) $($ADUser.SID)&quot;
        $NewUserPrincipalName =  &quot;$($ADUser.UserPrincipalName.split('@')[0]) $($ADUser.SID)@$($ADUser.UserPrincipalName.split('@')[1])&quot;
        $NewSamAccountName = ($($ADUser.SID).tostring() -replace &quot;-&quot;,&quot;&quot;).substring(20)

        Write-Host &quot;Rename Name $($ADUser.Name) to $NewName&quot;
        Rename-ADObject $ADUser -NewName $NewName

        Write-Host &quot;Rename UserPrincipalName $($ADUser.UserPrincipalName) to $NewUserPrincipalName&quot;
        Get-ADUser $ADUser.SamAccountName | Set-ADUser -UserPrincipalName $NewUserPrincipalName -Description &quot;archived&quot;

        Write-Host &quot;Rename SamAccountName $($ADUser.SamAccountName) to $NewSamAccountName&quot;
        Get-ADUser $ADUser.SamAccountName | Set-ADUser -SamAccountName $NewSamAccountName

        $NewPrimarySmtpAddress = &quot;$($ADUser.UserPrincipalName.split('@')[0])$($ADUser.SID)@$($ADUser.UserPrincipalName.split('@')[1])&quot; -replace &quot;-&quot;,&quot;&quot;
        $OldPrimarySmtpAddress = $Mailbox.PrimarySmtpAddress

        if($Mailbox.psObject.TypeNames -contains &quot;Deserialized.Microsoft.Exchange.Data.Directory.Management.RemoteMailbox&quot;){

            $NewRemoteRoutingAddress = &quot;$($Mailbox.RemoteRoutingAddress.split(&quot;@&quot;)[0])$($ADUser.SID)@$($Mailbox.RemoteRoutingAddress.split(&quot;@&quot;)[1])&quot; -replace &quot;-&quot;,&quot;&quot;
            $OldRemoteRoutingAddress = $Mailbox.RemoteRoutingAddress

            Get-RemoteMailbox $ADuser.Name | %{

                Write-Host &quot;Rename PrimarySmtpAddress for $($_.PrimarySmtpAddress) to $NewPrimarySmtpAddress&quot;
                Set-RemoteMailbox $_.Alias -PrimarySmtpAddress $NewPrimarySmtpAddress;

                Write-Host &quot;Rename RemoteRoutingAddress for $($_.RemoteRoutingAddress) to $NewRemoteRoutingAddress&quot;
                Set-RemoteMailbox $_.Alias -RemoteRoutingAddress $NewRemoteRoutingAddress

                Write-Host &quot;Remove default mail addresses $OldRemoteRoutingAddress, $PrimarySmtpAddress on $($_.Alias)&quot;
                Set-RemoteMailbox $_.Alias -EmailAddresses @{remove = $OldRemoteRoutingAddress, $OldPrimarySmtpAddress}
            }

        }elseif($Mailbox.psObject.TypeNames -contains &quot;Deserialized.Microsoft.Exchange.Data.Directory.Management.Mailbox&quot;){

            Get-Mailbox $ADuser.Name | %{

                Write-Host &quot;Rename PrimarySmtpAddress for $($_.PrimarySmtpAddress) to $NewPrimarySmtpAddress&quot;
                Set-Mailbox $_.Alias -PrimarySmtpAddress $NewPrimarySmtpAddress

                Write-Host &quot;Remove default mail addresses $PrimarySmtpAddress on $($Mailbox.Alias)&quot;
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
$PSSession = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri &quot;https://$ExchangeServer/PowerShell/&quot; -Authentication Kerberos

# import
Import-PSSession $PSSession -AllowClobber

$Mailboxes = Get-Mailbox
$RemoteMailboxes = Get-RemoteMailbox

# disable mailbox and remote mailbox
Get-ADUser -Filter{Enabled -eq $false} -Properties mail | where{$_.mail -ne $null} |
    %{$ADUser = $_; $Mailboxes | where{$_.Name -eq $ADuser.Name -and $_.HiddenFromAddressListsEnabled -eq $false -and $FilterRecipientTypeDetails -contains $_.RecipientTypeDetails}} |%{
        Write-host &quot;Hide mailbox $($_.Name) from address lists.&quot;;
        Set-Mailbox $_.Name -HiddenFromAddressListsEnabled:$true;
        Rename-ADUserAndMailbox -ADUser $ADUser -MailBox $_
    }

# disable remote mailbox
Get-ADUser -Filter{Enabled -eq $false} -Properties mail | where{$_.mail -ne $null} |
    %{$ADUser = $_; $RemoteMailboxes | where{$_.Name -eq $ADuser.Name -and $_.HiddenFromAddressListsEnabled -eq $false -and $FilterRecipientTypeDetails -contains $_.RecipientTypeDetails}} | %{
        Write-host &quot;Hide remotemailbox $($_.Name) from address lists.&quot;;
        Set-RemoteMailbox $_.Name -HiddenFromAddressListsEnabled:$true;
        Rename-ADUserAndMailbox -ADUser $ADUser -MailBox $_
}

# destroy pssession
Remove-PSSession $PSSession

if($error){
    Send-PPErrorReport -FileName &quot;activedirectory.mail.config.xml&quot; -ScriptName $MyInvocation.InvocationName
}

[/code]

Latest version of this script: <a href="https://gist.github.com/6780143" target="_blank">https://gist.github.com/6780143</a>

As always I recommand you to install my project <a href="https://github.com/janikvonrotz/Powershell-Profile" target="_blank">PowerShell Profile</a> to run this script properly.