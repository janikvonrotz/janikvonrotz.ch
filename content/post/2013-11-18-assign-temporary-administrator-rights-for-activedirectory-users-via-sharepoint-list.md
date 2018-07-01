---
id: 744
title: Assign Temporary Administrator Rights for ActiveDirectory Users via SharePoint list
date: 2013-11-18T16:55:32+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=744
permalink: /2013/11/18/assign-temporary-administrator-rights-for-activedirectory-users-via-sharepoint-list/
dsq_thread_id:
  - "1976601577"
image: /wp-content/uploads/2013/08/Active-Directory-Logo.png
categories:
  - Active Directory
  - PowerShell
  - SharePoint
tags:
  - activedirectory
  - administrator
  - assign
  - client
  - computer
  - list
  - rights
  - sharepoint
  - temporary
  - user
---
In my company the user only have user rights on their computers. As you should know you'll face many problems with this restriction.

Many users want to install third party software on their computers or add a printer at home. To reduce argues and make the user happy, I'll assign administrator rights for a temporary time.

Based on a predefined GPO and based on a list showing which user has administrator rights in a specified time period, my PowerShell script creates new temporary GPO to assign local administrator rights.

<!--more-->

After the specified time period the scripts deletes the temporary GPO in order to remove the local admin rights.

Here is the template of the predefined GPO:

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/11/Temporäre-Administratorrechte.png"><img class="aligncenter size-full wp-image-746" alt="Temporäre Administratorrechte" src="https://janikvonrotz.ch/wp-content/uploads/2013/11/Temporäre-Administratorrechte.png" width="794" height="900" /></a>

User "test" is placeholder for the real user.

The list containing the users is stored on SharePoint site:

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/11/Temporäre-Administratorrechte-1.png"><img class="aligncenter size-full wp-image-753" alt="Temporäre Administratorrechte 1" src="https://janikvonrotz.ch/wp-content/uploads/2013/11/Temporäre-Administratorrechte-1.png" width="772" height="250" /></a>

The scripts reads from the SharePoint list with predefined credentials and remote configuration file wich is part of my PowerShell Profile project (check out the requirements on the bottom).

Watch out to store the computer object in a OU instead of an container, it's not possible to assign GPOs to an AD container!

[code lang="ps"]
&lt;#
$Metadata = @{
	Title = &quot;Assign Temporary Administrator Rights&quot;
	Filename = &quot;Assign-TemporaryAdministratorRights.ps1&quot;
	Description = &quot;&quot;
	Tags = &quot;powershell, script, activedirectory, assign, temporary, administrator, rights, computer&quot;
	Project = &quot;&quot;
	Author = &quot;Janik von Rotz&quot;
	AuthorContact = &quot;https://janikvonrotz.ch&quot;
	CreateDate = &quot;2013-11-15&quot;
	LastEditDate = &quot;2013-11-18&quot;
	Url = &quot;&quot;
	Version = &quot;1.0.0&quot;
	License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit https://creativecommons.org/licenses/by-sa/3.0/ch/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#&gt;

try{
    #--------------------------------------------------#
    # modules
    #--------------------------------------------------#
    Import-Module ActiveDirectory
    Import-Module GroupPolicy

    #--------------------------------------------------#
    # settings
    #--------------------------------------------------#

    # var #Username# replaces username, var #Computername# replaces computername
    $GPOTemplate = &quot;Windows User #Username# - #Computername# Lokaler Administrator&quot;
    $TempFolder = &quot;C:export&quot;
    $SPWebUrl = (Get-SPUrl &quot;https://sharepoint.vbl.ch/finanzen/it/Abteilungssite/SitePages/Homepage.aspx&quot;).Url
    $SPListName = &quot;Temporäre Adminrechte&quot;
    $RemoteConnectionKey = &quot;sp1&quot;

    #--------------------------------------------------#
    # main
    #--------------------------------------------------#

    $Computer = Get-RemoteConnection -Name $RemoteConnectionKey
    $Credential = Import-PSCredential -Path (Get-ChildItem $PSconfigs.Path -Filter &quot;SharePoint.credential.config.xml&quot; -Recurse).FullName
    $Session = New-PSSession -ComputerName $Computer.Name -Credential $Credential -ConfigurationName microsoft.powershell
    $Computer.SnapIns | %{ Invoke-Command -Session $Session -ScriptBlock {param ($Name) Add-PSSnapin -Name $Name} -ArgumentList $_}
    [ScriptBlock]$ScriptBlock = [scriptblock]::Create(@&quot;
Get-SPWeb '$SPWebUrl' | %{
    `$_.Lists['$SPListName'].GetItems() | %{
        `$(New-Object PSObject -Property @{
            Mail = `$_[&quot;Title&quot;].toString()
            Computer = `$_[&quot;Computer&quot;].toString()
            From = `$_[&quot;From&quot;].toString()
            To = `$_[&quot;To&quot;].toString()
        })
    }
}
&quot;@)
    $Config = Invoke-Command -Session $Session -ScriptBlock $ScriptBlock
    Remove-PSSession $Session

    &lt;#
     $Config = @(
          $(New-Object PSObject -Property @{
              Mail = &quot;name.surname@domain.ch&quot;
              Computer = &quot;tpbmar1&quot;
              From = &quot;18.11.2013&quot;
              To = &quot;25.11.2013&quot;
          }),

          $(New-Object PSObject -Property @{
              Mail = &quot;name.surname@vbl.ch&quot;
              Computer = &quot;tpfit9&quot;
              From = &quot;15.11.2013&quot;
              To = &quot;21.11.2013&quot;
          }),
      )
    #&gt;
    $Config | %{

        # get settings
        $ADComputer = Get-ADComputer $_.Computer
        $ADUser = Get-ADUser -Filter &quot;mail -eq '$($_.Mail)'&quot; | select -first 1
        $GPOName = ($GPOTemplate -replace &quot;#Username#&quot;, $ADUser.Name -replace &quot;#Computername#&quot;, $ADComputer.Name)
        $SourceGPO = Get-GPO $GPOTemplate
        $TargetOU = $ADComputer.DistinguishedName -replace &quot;CN=$($ADComputer.Name),&quot;,&quot;&quot;
        $FromDate = Get-Date $_.From
        $ToDate = Get-Date $_.To
        $Date = $(Get-Date)

        # create temp folder
        if(-not (Test-Path $TempFolder)){New-Item -Path $TempFolder -ItemType Directory}

        # get gpo
        $GPO = Get-GPO -Name $GPOName -ErrorAction SilentlyContinue

        # create if not exist
        if(-not $GPO -and $Date -gt $FromDate -and $Date -lt $ToDate){

            # create new gpo
            $GPO = New-GPO -Name $GPOName
            $GPO | New-GPLink -Target $TargetOU
            $GPO | Set-GPPermissions -Replace -PermissionLevel None -TargetName &quot;Authentifizierte Benutzer&quot; -TargetType Group
            $GPO | Set-GPPermissions -PermissionLevel GpoApply -TargetName $ADComputer.Name -TargetType Computer

            # backup template gpo
            $GPOBackup = $SourceGPO | Backup-GPO -Path $TempFolder
            $PathToXML = Join-Path $TempFolder (&quot;{&quot; + $GPOBackup.Id + &quot;}DomainSysvolGPOMachinePreferencesGroupsGroups.xml&quot;)
            $PathToFolder = Join-Path $TempFolder (&quot;{&quot; + $GPOBackup.Id + &quot;}&quot;)
            [xml]$GroupXML = Get-Content $PathToXML

            # update template gpo settings
            $GroupXML.Groups.Group.Properties.Members.Member.name = $(Get-ADDomain).NetBIOSName + &quot;&quot; +$ADUser.SamAccountName
            $GroupXML.Groups.Group.Properties.Members.Member.sid = &quot;$($ADUser.SID)&quot;
            $GroupXML.Save($PathToXML)

            # import to new gpo
            Import-GPO -BackupId $GPOBackup.Id -TargetGuid $GPO.Id -path $TempFolder

            # clean up tempfolder
            Remove-Item $PathToFolder -Force -confirm:$false -Recurse

            Write-PPEventLog -Message &quot;Added temporary administrator rights for: $($_.Mail) on computer: $($_.Computer)&quot; -Source &quot;Assign Temporary Administrator Rights&quot; -WriteMessage

        # delete gpo
        }elseif($GPO -and $Date -gt $ToDate ){

           $GPO | Remove-GPO
           Write-PPEventLog -Message &quot;Removed temporary administrator rights for: $($_.Mail) on computer: $($_.Computer)&quot; -Source &quot;Assign Temporary Administrator Rights&quot; -WriteMessage
        }
    }
}catch{

    Write-PPErrorEventLog -Source &quot;Assign Temporary Administrator Rights&quot;
}
[/code]

Run this script as a hourly or daily scheduled task.

Latest version of this script: <a href="https://gist.github.com/7487228" target="_blank">https://gist.github.com/7487228</a>

Requirements to run this script:

<ul>
    <li><a href="https://github.com/janikvonrotz/PowerShell-Profile" target="_blank">PowerShell Profile</a></li>
</ul>