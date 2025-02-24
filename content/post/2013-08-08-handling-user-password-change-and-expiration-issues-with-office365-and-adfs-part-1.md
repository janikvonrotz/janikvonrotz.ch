---
title: 'Handling user password change and expiration issues with Office365 and ADFS - Part 1'
date: 2013-08-08T14:41:37+00:00
author: Janik von Rotz
slug: handling-user-password-change-and-expiration-issues-with-office365-and-adfs-part-1
images:
  - /wp-content/uploads/2013/08/microsoft-office-365-e1394705447131.jpg
categories:
  - Office 365
  - scripting
tags:
  - activedirectory
  - expiration
  - federation
  - mail
  - office365
  - password
  - service
---
Recently I've setup a Office365 Service with ADFS (Active Directory Federation Service) and a DirSync Server.

Sadly I forgot about a huge disadvantage in this architecture, due to using ADFS as an authentication provider, it's not possible to change a users password. The communication form the local ActiveDirectory environment to the cloud based Office365 services is only one directional.

That's why there are only 2 options yet to handle the user password change and expiration:

<ol>
    <li>Disable the users password expiration or</li>
    <li>Set up a enterprise connected platform to deal with the password change and expiration.</li>
</ol>

At this time option 1 is active in my environment and option 2 is my goal.

In this post series want to show you the solution I've developed.

Let's start with password expiration. Because Office365 doesn't handle password expiration, that's why I have to use another channel to show the users on which date their passwords expire:  Let's do it with an bulk e-mail job.

<!--more-->

<h1>Password Expiration E-Mail Job</h1>

This part isn't a lot of effort. There are 2 requirements, one is a must and the other is optional:

<ul>
    <li>Active Directory Module for PowerShell</li>
    <li>To create scheduled job on your windows server I've built the function `Add-SheduledTask` which is part of my<a href="https://github.com/janikvonrotz/Powershell-Profile" target="_blank"> PowerShell Profile project</a>, you can either<a href="https://github.com/janikvonrotz/Powershell-Profile/blob/master/functions/Windows/Add-SheduledTask.ps1" target="_blank"> download the function</a> or <a href="https://github.com/janikvonrotz/Powershell-Profile#readme" target="_blank">install this project</a> on your windows server.</li>
</ul>

Now the script:

```powershell
<#
$Metadata = @{
	Title = "Send Password Expiration Reminder"
	Filename = "Send-PasswordExpirationReminder.ps1"
	Description = ""
	Tags = "powershell, script, jobs"
	Project = ""
	Author = "Janik von Rotz"
	AuthorContact = "http://.janikvonrotz.ch"
	CreateDate = "2013-08-08"
	LastEditDate = "2013-11-25"
	Version = "2.1.0"
	License = @'
This work is licensed under the Creative Commons Attribution-NonCommercial-NoDerivs 3.0 Unported License.
To view a copy of this license, visit http://creativecommons.org/licenses/by-nc-nd/3.0/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#>

try{

    #--------------------------------------------------#
    # modules
    #--------------------------------------------------#    
    Import-Module ActiveDirectory
	
    #--------------------------------------------------#
    # settings
    #--------------------------------------------------#   
    $TriggerDays = 25, 10, 5, 1
    $SendLinkOnDays = 25,10, 5, 1
	$DaysBeforeDisablingUsersWithPasswordNeverExpires = 180
	$ADGroup = "S-1-5-21-1744926098-708661255-2033415169-36648" # Memberof GroupName should be "SPO_PasswordNotification"   
    
    #--------------------------------------------------#
    # main
    #--------------------------------------------------#

    # get mail config         
    $Mail = Get-PPConfiguration $PSconfigs.Mail.Filter | %{$_.Content.Mail | where{$_.Name -eq "PasswordReminder"}} | select -first 1

    # get days until password expires
    $MaxDays = (Get-ADDefaultDomainPasswordPolicy).MaxPasswordAge.Days 
    if($MaxDays -le 0){throw "Domain 'MaximumPasswordAge' password policy is not configured."}

    # Set days when an email should be sent to inform the users
    $TriggerDays = 25, 10, 5, 1
    $SendLinkOnDays = 25,10, 5, 1

    foreach($TriggerDay in $TriggerDays){    
    
        # Memberof GroupName should be "SPO_PasswordNotification"       
        Get-ADGroupMember $ADGroup -Recursive | 
        Get-ADUser -Properties Enabled, lastLogonTimestamp, PasswordNeverExpires, PasswordLastSet, Mail, DisplayName |
        Select *, @{L = "PasswordExpires";E = { 
            if($_.PasswordNeverExpires){
                $DaysBeforeDisablingUsersWithPasswordNeverExpires - ((Get-Date) - ($_.PasswordLastSet)).Days
            }else{
                $MaxDays - ((Get-Date) - ($_.PasswordLastSet)).Days
            }
        }} |
        where{($_.Enabled -eq $true) -and ($_.PasswordExpires -eq $TriggerDay)} | %{ 
                              
            # set subject
            $Subject = "Passwort Erinnerung: $($_.DisplayName) ihr Passwort läuft in $($_.PasswordExpires) Tagen ab"
            
            $BodyFont = "font-size: 11pt; font-family: Calibri"
            
            # create mail message
            $Body = "<p style = ""$BodyFont"">Guten Tag $($_.DisplayName) <br/> <br/> Ihr Passwort läuft am $(Get-Date (Get-Date).AddDays($_.PasswordExpires) -Format D) ab.</b></p>"          
            if($SendLinkOnDays -contains $TriggerDay){            
                $Body += "<p style = ""$BodyFont"">Bitte ändern Sie das Passwort bevor es abläuft. Rufen Sie dazu die folgende Seite auf: <a href=""https://vbluzern.sharepoint.com/Support/_layouts/15/start.aspx#/SitePages/Passwortwechsel.aspx"" target=""_blank"">Link</a></p>"
            }
             $Body += "<p style = ""$BodyFont"">ACHTUNG! Dieses E-Mail wurde von einem unbeaufsichtigtem Konto verschickt, Antworten an den Sender dieser E-Mail werden nicht bearbeitet.</p>"

            # send mail
            Write-PPEventLog "$($MyInvocation.InvocationName)`n`nSend password reminder to $($_.Mail)" -WriteMessage -Source "Send Password Expiration Reminder" 
            Send-MailMessage -To $_.Mail -From $mail.FromAddress -Subject $Subject -Body $Body -SmtpServer $Mail.OutSmtpServer -BodyAsHtml -Priority High -Encoding ([System.Text.Encoding]::UTF8)
        
        }        
    }
   
}catch{

	Write-PPErrorEventLog -Source "Send Password Expiration Reminder" -ClearErrorVariable
}
```

[https://gist.github.com/6184059](https://gist.github.com/6184059)

For the memberof Group I recommend to use the SID instead of the DN. I'll show you how the get the SID:

```powershell

PS C:Userssa-spadmin> (Get-QADGroup "SPO_PasswordNotification").Sid

  BinaryLength AccountDomainSid                                                               Value
  ------------ ----------------                                                               -----
			28 S-1-5-21-1744926098-708661255-2033415169                                       S-1-5-21-1744926098-708661255-2033415169-36648

PS C:Userssa-spadmin> Get-QADUser -MemberOf "S-1-5-21-1744926098-708661255-2033415169-36648"

```

<h1>Update</h1>

Part two: <a href="https://janikvonrotz.ch/2013/09/23/handling-user-password-change-and-expiration-issues-withoffice365-and-adfs-part-2/">https://janikvonrotz.ch/2013/09/23/handling-user-password-change-and-expiration-issues-withoffice365-and-adfs-part-2/</a>