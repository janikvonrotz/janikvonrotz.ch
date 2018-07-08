---
title: Finally! Manage Exchange mailbox permissions with Active Directory groups
date: 2014-01-27T19:04:37+00:00
author: Janik von Rotz
permalink: /2014/01/27/finally-manage-exchange-mailbox-permissions-with-active-directory-groups/
dsq_thread_id:
  - "2181973874"
image: /wp-content/uploads/2013/08/exchange-2013-e1393417827333.jpg
categories:
  - Exchange
tags:
  - active
  - directory
  - exchange
  - groups
  - mailbox
  - permission
  - synchronize
  - task
---
I've tried many ways to assign permissions for an Active Directory group on a Exchange (2010) mailbox, but it's simply not possible.

Fortunately nothing's impossible with PowerShell.

The following script can handle this issue by:

<!--more-->

<ul>
    <li>Creating an Active Directory permission groups linked to the service mailbox for each permission type on each mailbox on the Exchange server filtered by Active Directory users and groups holding <strong>not</strong> service mailbox users.</li>
    <li>Synchronize members for each mailbox permission type.</li>
    <li>Delete unused permission groups.</li>
    <li>Auditing events and error with <a href="https://github.com/janikvonrotz/PowerShell-Profile">PowerShell Profile</a>.</li>
</ul>

[![Synchronize Service Mailbox Access Groups](/wp-content/uploads/2014/01/Synchronize-Service-Mailbox-Access-Groups-1024x413.jpg)](/wp-content/uploads/2014/01/Synchronize-Service-Mailbox-Access-Groups.jpg)

```powershell
<#
$Metadata = @{
	Title = "Synchronize Service Mailbox Access Groups"
	Filename = "Sync-ServiceMailboxAccessGroups.ps1"
	Description = ""
	Tags = "powershell, activedirectory, exchange, synchronization, access, mailbox, groups, permissions"
	Project = ""
	Author = "Janik von Rotz"
	AuthorContact = "https://janikvonrotz.ch"
	CreateDate = "2014-01-27"
	LastEditDate = "2014-01-27"
	Url = ""
	Version = "0.0.0"
	License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit https://creativecommons.org/licenses/by-sa/3.0/ch/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#>

Import-Module ActiveDirectory

# Connect Exchange Server
$ExchangeServer = (Get-RemoteConnection ex1).Name
$PSSession = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri "https://$ExchangeServer/PowerShell/" -Authentication Kerberos
Import-PSSession $PSSession

$Config = @{
    OU = "OU=Mailbox,OU=Exchange,OU=Services,OU=vblusers2,DC=vbl,DC=ch"

    ADGroupFilter = @{
        NamePrefix = "EX_"
        NameInfix = ""
        NameSuffix = ""
    }

    ADGroup = @{
        NamePrefix = "EX_"
        PermissionSeperator = "#"
    }

    ADGroupMailboxReferenceAttribute = "extensionAttribute1"

    MailBoxFilter = @{
        RecipientTypeDetails = "UserMailbox"
        ADGroupsAndUsers = "F_Mitarbeiter","F_Archivierte Benutzer"
        ExcludeDisplayName = "FederatedEmail.4c1f4d8b-8179-4148-93bf-00a95fa1e042"
    }

} | %{New-Object PSObject -Property $_}

# get domain name
$Domain = "$(((Get-ADDomain).Name).toupper())"

# get ad user objects
$Config.MailBoxFilter.ADGroupsAndUsers = $Config.MailBoxFilter.ADGroupsAndUsers | %{
    Get-ADObject -Filter {(Name -eq $_) -or (ObjectGUID -eq $_)} | %{
        if($_.ObjectClass -eq "user"){$_.DistinguishedName
        }elseif($_.ObjectClass -eq "group"){ Get-ADGroupMember $_.DistinguishedName -Recursive}
    } | Get-ADUser -Properties Mail
}

# create mail list to filter mailboxes
$Config.MailboxFilter.AllowedMails = $Config.MailBoxFilter.ADGroupsAndUsers | %{"$($_.Mail)"}

# create SamAccountName list to filter mailbox permissions
$Config.ADGroupFilter.AllowedUsers = $Config.MailboxFilter.ADGroupsAndUsers | %{"$($Domain + $_.SamAccountName)"}

# get exisiting mailbox permission ad groups
$ADGroups = Get-ADGroup -Filter * -SearchBase $Config.OU -Properties $Config.ADGroupMailboxReferenceAttribute | where{$_.Name.StartsWith($Config.ADGroupFilter.NamePrefix) -and $_.Name.Contains($Config.ADGroupFilter.NameInfix) -and $_.Name.EndsWith($Config.ADGroupFilter.NameSuffix)}

# sync mailbox access groups for each mailbox
$Mailboxes = Get-Mailbox | where{$_.RecipientTypeDetails -eq $Config.MailBoxFilter.RecipientTypeDetails -and $Config.MailboxFilter.AllowedMails -notcontains $_.PrimarySmtpAddress.tolower() -and $Config.MailBoxFilter.ExcludeDisplayName -notcontains $_.DisplayName}

$RequiredADGroups = $Mailboxes | %{

    # set variables
    $ADPermissionGroups = @()
    $Mailbox = $_

    Write-Host "Parsing permissions on mailbox: $($_.Alias)"

    # get existing permission groups
    $ADPermissionGroups = $ADGroups | where{(iex "`$_.$($Config.ADGroupMailboxReferenceAttribute)") -eq $Mailbox.Guid}
    $ADExistingPermissionGroupTypes = $ADPermissionGroups| where{$_} | %{$_.Name.split("#")[1]}

    # get existing and allowed mailbox permissions
    $MailboxPermissions = $Mailbox | Get-MailboxPermission | where{$Config.ADGroupFilter.AllowedUsers -contains $_.User}

    # create an ad group foreach permissiontype that is required
    $NewADPermissionGroupTypes = $MailboxPermissions | %{"$($_.AccessRights)".split(", ") | where{$_} | %{$_} | %{$_ | where{$ADExistingPermissionGroupTypes -notcontains $_}}}
    $NewADPermissionGroupTypes | Group | %{

        # create ad group name foreach permission type
        $Name = $config.ADGroup.NamePrefix + $Mailbox.Displayname + $Config.ADGroup.PermissionSeperator + $_.Name

        Write-PPEventLog -Message "Add service mailbox access group: $Name" -Source "Synchronize Service Mailbox Access Groups" -WriteMessage
        New-ADGroup -Name $Name -SamAccountName $Name -GroupCategory Security -GroupScope Global -DisplayName $Name  -Path $Config.OU -Description "Exchange Access Group for: $($Mailbox.Displayname)"
        Get-ADGroup $Name

    } | %{

        # set the reference from the adgroup to the mailbox
        iex "`$_.$($Config.ADGroupMailboxReferenceAttribute) = `"$($Mailbox.Guid)`""
        Set-ADGroup -Instance $_
        $ADGroup = $_

        # add existing members to the permission group
        $MailboxPermissions | where{$_.AccessRights -match $ADGroup.Name.split("#")[1]} | %{
            Add-ADGroupMember -Identity $ADGroup -Members ($_.User -replace "$Domain","")
        }

        # output ad permission groups
        $_
    }

    # check members foreach permission group
    $ADPermissionGroups | where{$_} | %{

        # get permission type
        $Permission = $_.Name.split("#")[1]

        # get existin ad user groups
        $ADPermissionGroupUsers = $_ | Get-ADGroupMember -Recursive | select @{L="User";E={$($Domain + $_.SamAccountName)}}
        $MailboxUsers =  $MailboxPermissions | where{$_.AccessRights -match $Permission} | select user

        # compare these groups and update members
        if($ADPermissionGroupUsers){
            $PermissionDiff = Compare-Object -ReferenceObject $ADPermissionGroupUsers -DifferenceObject $MailboxUsers -Property User

            # add member
            $PermissionDiff | where{$_.SideIndicator -eq "<="} | %{

                Write-PPEventLog -Message "Add mailbox permission: $Permission for user: $($_.User) on mailbox: $($Mailbox.Alias)" -Source "Synchronize Service Mailbox Access Groups" -WriteMessage
                Add-MailboxPermission -Identity $Mailbox.Alias -User $_.User -AccessRights $Permission
            }

            # remove member
            $PermissionDiff | where{$_.SideIndicator -eq "=>"} | %{

                Write-PPEventLog -Message "Remove mailbox permission: $Permission for user: $($_.User) on mailbox: $($Mailbox.Alias)" -Source "Synchronize Service Mailbox Access Groups" -WriteMessage
                Remove-MailboxPermission -Identity $Mailbox.Alias -User $_.User -AccessRights $Permission -Confirm:$false
            }
        }

        #output ad permission group to delete old ad permission groups
        $_
    }
}

$ADGroups | where{$RequiredADGroups -notcontains $_} | %{

    Write-PPEventLog -Message "Remove service mailbox access group: $Name" -Source "Synchronize Service Mailbox Access Groups" -WriteMessage
    Remove-ADGroup -Identity $_ -Confirm:$false
}

Remove-PSSession $PSSession
Write-PPErrorEventLog -Source "Synchronize Service Mailbox Access Groups" -ClearErrorVariable
```

Latest version of this script: <a href="https://gist.github.com/8653473" target="_blank">https://gist.github.com/8653473</a>

<h1>Requirements</h1>

To run this script properly and use commands as `Write-PPEventLog` I recommend you to install my project <a href="https://github.com/janikvonrotz/PowerShell-Profile">PowerShell Profile</a>.