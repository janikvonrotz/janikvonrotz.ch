---
id: 1019
title: Finally! Manage Exchange mailbox permissions with Active Directory groups
date: 2014-01-27T19:04:37+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=1019
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

<a href="https://janikvonrotz.ch/wp-content/uploads/2014/01/Synchronize-Service-Mailbox-Access-Groups.jpg"><img class="aligncenter size-large wp-image-1020" alt="Synchronize Service Mailbox Access Groups" src="https://janikvonrotz.ch/wp-content/uploads/2014/01/Synchronize-Service-Mailbox-Access-Groups-1024x413.jpg" width="474" height="191" /></a>

[code lang="ps"]
&lt;#
$Metadata = @{
	Title = &quot;Synchronize Service Mailbox Access Groups&quot;
	Filename = &quot;Sync-ServiceMailboxAccessGroups.ps1&quot;
	Description = &quot;&quot;
	Tags = &quot;powershell, activedirectory, exchange, synchronization, access, mailbox, groups, permissions&quot;
	Project = &quot;&quot;
	Author = &quot;Janik von Rotz&quot;
	AuthorContact = &quot;https://janikvonrotz.ch&quot;
	CreateDate = &quot;2014-01-27&quot;
	LastEditDate = &quot;2014-01-27&quot;
	Url = &quot;&quot;
	Version = &quot;0.0.0&quot;
	License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit https://creativecommons.org/licenses/by-sa/3.0/ch/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#&gt;

Import-Module ActiveDirectory

# Connect Exchange Server
$ExchangeServer = (Get-RemoteConnection ex1).Name
$PSSession = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri &quot;https://$ExchangeServer/PowerShell/&quot; -Authentication Kerberos
Import-PSSession $PSSession

$Config = @{
    OU = &quot;OU=Mailbox,OU=Exchange,OU=Services,OU=vblusers2,DC=vbl,DC=ch&quot;

    ADGroupFilter = @{
        NamePrefix = &quot;EX_&quot;
        NameInfix = &quot;&quot;
        NameSuffix = &quot;&quot;
    }

    ADGroup = @{
        NamePrefix = &quot;EX_&quot;
        PermissionSeperator = &quot;#&quot;
    }

    ADGroupMailboxReferenceAttribute = &quot;extensionAttribute1&quot;

    MailBoxFilter = @{
        RecipientTypeDetails = &quot;UserMailbox&quot;
        ADGroupsAndUsers = &quot;F_Mitarbeiter&quot;,&quot;F_Archivierte Benutzer&quot;
        ExcludeDisplayName = &quot;FederatedEmail.4c1f4d8b-8179-4148-93bf-00a95fa1e042&quot;
    }

} | %{New-Object PSObject -Property $_}

# get domain name
$Domain = &quot;$(((Get-ADDomain).Name).toupper())&quot;

# get ad user objects
$Config.MailBoxFilter.ADGroupsAndUsers = $Config.MailBoxFilter.ADGroupsAndUsers | %{
    Get-ADObject -Filter {(Name -eq $_) -or (ObjectGUID -eq $_)} | %{
        if($_.ObjectClass -eq &quot;user&quot;){$_.DistinguishedName
        }elseif($_.ObjectClass -eq &quot;group&quot;){ Get-ADGroupMember $_.DistinguishedName -Recursive}
    } | Get-ADUser -Properties Mail
}

# create mail list to filter mailboxes
$Config.MailboxFilter.AllowedMails = $Config.MailBoxFilter.ADGroupsAndUsers | %{&quot;$($_.Mail)&quot;}

# create SamAccountName list to filter mailbox permissions
$Config.ADGroupFilter.AllowedUsers = $Config.MailboxFilter.ADGroupsAndUsers | %{&quot;$($Domain + $_.SamAccountName)&quot;}

# get exisiting mailbox permission ad groups
$ADGroups = Get-ADGroup -Filter * -SearchBase $Config.OU -Properties $Config.ADGroupMailboxReferenceAttribute | where{$_.Name.StartsWith($Config.ADGroupFilter.NamePrefix) -and $_.Name.Contains($Config.ADGroupFilter.NameInfix) -and $_.Name.EndsWith($Config.ADGroupFilter.NameSuffix)}

# sync mailbox access groups for each mailbox
$Mailboxes = Get-Mailbox | where{$_.RecipientTypeDetails -eq $Config.MailBoxFilter.RecipientTypeDetails -and $Config.MailboxFilter.AllowedMails -notcontains $_.PrimarySmtpAddress.tolower() -and $Config.MailBoxFilter.ExcludeDisplayName -notcontains $_.DisplayName}

$RequiredADGroups = $Mailboxes | %{

    # set variables
    $ADPermissionGroups = @()
    $Mailbox = $_

    Write-Host &quot;Parsing permissions on mailbox: $($_.Alias)&quot;

    # get existing permission groups
    $ADPermissionGroups = $ADGroups | where{(iex &quot;`$_.$($Config.ADGroupMailboxReferenceAttribute)&quot;) -eq $Mailbox.Guid}
    $ADExistingPermissionGroupTypes = $ADPermissionGroups| where{$_} | %{$_.Name.split(&quot;#&quot;)[1]}

    # get existing and allowed mailbox permissions
    $MailboxPermissions = $Mailbox | Get-MailboxPermission | where{$Config.ADGroupFilter.AllowedUsers -contains $_.User}

    # create an ad group foreach permissiontype that is required
    $NewADPermissionGroupTypes = $MailboxPermissions | %{&quot;$($_.AccessRights)&quot;.split(&quot;, &quot;) | where{$_} | %{$_} | %{$_ | where{$ADExistingPermissionGroupTypes -notcontains $_}}}
    $NewADPermissionGroupTypes | Group | %{

        # create ad group name foreach permission type
        $Name = $config.ADGroup.NamePrefix + $Mailbox.Displayname + $Config.ADGroup.PermissionSeperator + $_.Name

        Write-PPEventLog -Message &quot;Add service mailbox access group: $Name&quot; -Source &quot;Synchronize Service Mailbox Access Groups&quot; -WriteMessage
        New-ADGroup -Name $Name -SamAccountName $Name -GroupCategory Security -GroupScope Global -DisplayName $Name  -Path $Config.OU -Description &quot;Exchange Access Group for: $($Mailbox.Displayname)&quot;
        Get-ADGroup $Name

    } | %{

        # set the reference from the adgroup to the mailbox
        iex &quot;`$_.$($Config.ADGroupMailboxReferenceAttribute) = `&quot;$($Mailbox.Guid)`&quot;&quot;
        Set-ADGroup -Instance $_
        $ADGroup = $_

        # add existing members to the permission group
        $MailboxPermissions | where{$_.AccessRights -match $ADGroup.Name.split(&quot;#&quot;)[1]} | %{
            Add-ADGroupMember -Identity $ADGroup -Members ($_.User -replace &quot;$Domain&quot;,&quot;&quot;)
        }

        # output ad permission groups
        $_
    }

    # check members foreach permission group
    $ADPermissionGroups | where{$_} | %{

        # get permission type
        $Permission = $_.Name.split(&quot;#&quot;)[1]

        # get existin ad user groups
        $ADPermissionGroupUsers = $_ | Get-ADGroupMember -Recursive | select @{L=&quot;User&quot;;E={$($Domain + $_.SamAccountName)}}
        $MailboxUsers =  $MailboxPermissions | where{$_.AccessRights -match $Permission} | select user

        # compare these groups and update members
        if($ADPermissionGroupUsers){
            $PermissionDiff = Compare-Object -ReferenceObject $ADPermissionGroupUsers -DifferenceObject $MailboxUsers -Property User

            # add member
            $PermissionDiff | where{$_.SideIndicator -eq &quot;&lt;=&quot;} | %{

                Write-PPEventLog -Message &quot;Add mailbox permission: $Permission for user: $($_.User) on mailbox: $($Mailbox.Alias)&quot; -Source &quot;Synchronize Service Mailbox Access Groups&quot; -WriteMessage
                Add-MailboxPermission -Identity $Mailbox.Alias -User $_.User -AccessRights $Permission
            }

            # remove member
            $PermissionDiff | where{$_.SideIndicator -eq &quot;=&gt;&quot;} | %{

                Write-PPEventLog -Message &quot;Remove mailbox permission: $Permission for user: $($_.User) on mailbox: $($Mailbox.Alias)&quot; -Source &quot;Synchronize Service Mailbox Access Groups&quot; -WriteMessage
                Remove-MailboxPermission -Identity $Mailbox.Alias -User $_.User -AccessRights $Permission -Confirm:$false
            }
        }

        #output ad permission group to delete old ad permission groups
        $_
    }
}

$ADGroups | where{$RequiredADGroups -notcontains $_} | %{

    Write-PPEventLog -Message &quot;Remove service mailbox access group: $Name&quot; -Source &quot;Synchronize Service Mailbox Access Groups&quot; -WriteMessage
    Remove-ADGroup -Identity $_ -Confirm:$false
}

Remove-PSSession $PSSession
Write-PPErrorEventLog -Source &quot;Synchronize Service Mailbox Access Groups&quot; -ClearErrorVariable
[/code]

Latest version of this script: <a href="https://gist.github.com/8653473" target="_blank">https://gist.github.com/8653473</a>

<h1>Requirements</h1>

To run this script properly and use commands as <code>Write-PPEventLog</code> I recommend you to install my project <a href="https://github.com/janikvonrotz/PowerShell-Profile">PowerShell Profile</a>.