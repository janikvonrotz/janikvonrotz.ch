---
id: 398
title: Manage Users in ActiveDirectory with PowerShell
date: 2013-08-09T10:18:06+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=398
permalink: /2013/08/09/manage-users-in-activedirectory-with-powershell/
dsq_thread_id:
  - "1588015157"
image: /wp-content/uploads/2013/08/Active-Directory-Logo.png
categories:
  - Active Directory
  - PowerShell
tags:
  - activedirectory
  - management
  - powershell
  - quest
  - user
---
The PowerShell ActiveDirectory modules from Microsoft are definitely a pain. That's why Quest (Dell) has developed a bunch of <a href="https://www.quest.com/powershell/activeroles-server.aspx">CMDlets </a>to make the user management through PowerShell a lot easier.

I would like to show you how I create and update my AD users.

As base there's always a CSV file like this one: <a href="https://janikvonrotz.ch/wp-content/uploads/2013/08/TemplateADUsers.csv">TemplateADUsers</a>

<!--more-->

<h1>Create new users</h1>

To a new user I'll use this script:

[code lang="ps" highlight="1,3"]

Import-Module Quest.ActiveRoles.ArsPowerShellSnapIn

$ADUsers = Import-CSV "ADUsers.csv" -Delimiter ";"
$ADUsers | foreach{
    if(!(Get-QADUser $_.Name)){

        Write-Host "Creating User" $_.Name

        # create a new user
        New-QADUser -Name $_.Name `
            -DisplayName $_.DisplayName `
            -FirstName $_.givenName `
            -LastName $_.LastName `
            -Company $_.company `
            -PostalCode $_.postalCode `
            -PostOfficeBox $_.postOfficeBox `
            -StreetAddress $_.streetAddress `
            -WebPage $_.wWWHomePage `
            -City $_.l `
            -Department $_.department `
            -Manager $_.manager `
            -Title $_.title `
            -Email $_.mail `
            -UserPrincipalName $_.mail `
            -ParentContainer $_.OU `
            -samAccountName $_.samAccountName `
            -UserPassword $_.Password `
            -ObjectAttributes @{
                extensionAttribute1= $_.extensionAttribute1;
                extensionAttribute2 = $_.extensionAttribute2;
                extensionAttribute3 = $_.extensionAttribute3;
                co = $_.co
                c = $_.c

            }` |
        Set-QADUser -PasswordNeverExpires $true |
        Out-Null

        # update group memberships
        $ADMemberOfs = $_.MemberOf -split ","
        foreach( $ADMemberOf in  $ADMemberOfs){
            Write-Host "Add User $($_.Name) to the group $ADMemberOf"
            Add-QADMemberOf $_.Name -Group $ADMemberOf | Out-Null
        }

    }else{

        Write-Host $_.Name "Already exist"

    }
}

[/code]

<h1>Update users</h1>

To update the users attributes I made this handy script:

[code lang="ps" highlight="1,3"]

Import-Module Quest.ActiveRoles.ArsPowerShellSnapIn

$ADUsers = Import-CSV "ADUsers.csv" -Delimiter ";"
$ADUsers | foreach{
    if((Get-QADUser $_.Name)){

        <# change Attributes

        Write-Host "Updating User" $_.Name
        Set-QADUser -Identity $_.Name `
            -Company $_.company `
            | Out-Null

        #>

        # update group memberships
        $ADMemberOfs = $_.MemberOf -split ","
        foreach( $ADMemberOf in  $ADMemberOfs){
            Write-Host "Add User $($_.Name) to the group $ADMemberOf"
            Add-QADMemberOf $_.Name -Group $ADMemberOf | Out-Null
        }

    }else{

        Write-Host $_.Name "doesnt' exist"

    }
}

[/code]

<ol>
    <li>Watch out! there are several ways to import the quest modules in PowerShell.</li>
    <li>Change the file path.</li>
</ol>