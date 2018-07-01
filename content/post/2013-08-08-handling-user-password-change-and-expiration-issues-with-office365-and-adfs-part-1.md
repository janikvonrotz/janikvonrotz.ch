---
id: 371
title: 'Handling user password change and expiration issues with Office365 and ADFS &#8211; Part 1'
date: 2013-08-08T14:41:37+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=371
permalink: /2013/08/08/handling-user-password-change-and-expiration-issues-with-office365-and-adfs-part-1/
dsq_thread_id:
  - "1585056725"
image: /wp-content/uploads/2013/08/microsoft-office-365-e1394705447131.jpg
categories:
  - Office 365
  - PowerShell
tags:
  - activedirectory
  - change
  - expiration
  - federation
  - mail
  - management
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
    <li>To create scheduled job on your windows server I've built the function <code>Add-SheduledTask</code> which is part of my<a href="https://github.com/janikvonrotz/Powershell-Profile" target="_blank"> PowerShell Profile project</a>, you can either<a href="https://github.com/janikvonrotz/Powershell-Profile/blob/master/functions/Windows/Add-SheduledTask.ps1" target="_blank"> download the function</a> or <a href="https://github.com/janikvonrotz/Powershell-Profile#readme" target="_blank">install this project</a> on your windows server.</li>
</ul>

Now the script:

[code language="ps"]
&lt;#
$Metadata = @{
	Title = &quot;Send Password Expiration Reminder&quot;
	Filename = &quot;Send-PasswordExpirationReminder.ps1&quot;
	Description = &quot;&quot;
	Tags = &quot;powershell, script, jobs&quot;
	Project = &quot;&quot;
	Author = &quot;Janik von Rotz&quot;
	AuthorContact = &quot;http://.janikvonrotz.ch&quot;
	CreateDate = &quot;2013-08-08&quot;
	LastEditDate = &quot;2013-11-25&quot;
	Version = &quot;2.1.0&quot;
	License = @'
This work is licensed under the Creative Commons Attribution-NonCommercial-NoDerivs 3.0 Unported License.
To view a copy of this license, visit http://creativecommons.org/licenses/by-nc-nd/3.0/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#&gt;

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
	$ADGroup = &quot;S-1-5-21-1744926098-708661255-2033415169-36648&quot; # Memberof GroupName should be &quot;SPO_PasswordNotification&quot;   
    
    #--------------------------------------------------#
    # main
    #--------------------------------------------------#

    # get mail config         
    $Mail = Get-PPConfiguration $PSconfigs.Mail.Filter | %{$_.Content.Mail | where{$_.Name -eq &quot;PasswordReminder&quot;}} | select -first 1

    # get days until password expires
    $MaxDays = (Get-ADDefaultDomainPasswordPolicy).MaxPasswordAge.Days 
    if($MaxDays -le 0){throw &quot;Domain 'MaximumPasswordAge' password policy is not configured.&quot;}

    # Set days when an email should be sent to inform the users
    $TriggerDays = 25, 10, 5, 1
    $SendLinkOnDays = 25,10, 5, 1

    foreach($TriggerDay in $TriggerDays){    
    
        # Memberof GroupName should be &quot;SPO_PasswordNotification&quot;       
        Get-ADGroupMember $ADGroup -Recursive | 
        Get-ADUser -Properties Enabled, lastLogonTimestamp, PasswordNeverExpires, PasswordLastSet, Mail, DisplayName |
        Select *, @{L = &quot;PasswordExpires&quot;;E = { 
            if($_.PasswordNeverExpires){
                $DaysBeforeDisablingUsersWithPasswordNeverExpires - ((Get-Date) - ($_.PasswordLastSet)).Days
            }else{
                $MaxDays - ((Get-Date) - ($_.PasswordLastSet)).Days
            }
        }} |
        where{($_.Enabled -eq $true) -and ($_.PasswordExpires -eq $TriggerDay)} | %{ 
                              
            # set subject
            $Subject = &quot;Passwort Erinnerung: $($_.DisplayName) ihr Passwort läuft in $($_.PasswordExpires) Tagen ab&quot;
            
            $BodyFont = &quot;font-size: 11pt; font-family: Calibri&quot;
            
            # create mail message
            $Body = &quot;&lt;p style = &quot;&quot;$BodyFont&quot;&quot;&gt;Guten Tag $($_.DisplayName) &lt;br/&gt; &lt;br/&gt; Ihr Passwort läuft am $(Get-Date (Get-Date).AddDays($_.PasswordExpires) -Format D) ab.&lt;/b&gt;&lt;/p&gt;&quot;          
            if($SendLinkOnDays -contains $TriggerDay){            
                $Body += &quot;&lt;p style = &quot;&quot;$BodyFont&quot;&quot;&gt;Bitte ändern Sie das Passwort bevor es abläuft. Rufen Sie dazu die folgende Seite auf: &lt;a href=&quot;&quot;https://vbluzern.sharepoint.com/Support/_layouts/15/start.aspx#/SitePages/Passwortwechsel.aspx&quot;&quot; target=&quot;&quot;_blank&quot;&quot;&gt;Link&lt;/a&gt;&lt;/p&gt;&quot;
            }
             $Body += &quot;&lt;p style = &quot;&quot;$BodyFont&quot;&quot;&gt;ACHTUNG! Dieses E-Mail wurde von einem unbeaufsichtigtem Konto verschickt, Antworten an den Sender dieser E-Mail werden nicht bearbeitet.&lt;/p&gt;&quot;

            # send mail
            Write-PPEventLog &quot;$($MyInvocation.InvocationName)`n`nSend password reminder to $($_.Mail)&quot; -WriteMessage -Source &quot;Send Password Expiration Reminder&quot; 
            Send-MailMessage -To $_.Mail -From $mail.FromAddress -Subject $Subject -Body $Body -SmtpServer $Mail.OutSmtpServer -BodyAsHtml -Priority High -Encoding ([System.Text.Encoding]::UTF8)
        
        }        
    }
   
}catch{

	Write-PPErrorEventLog -Source &quot;Send Password Expiration Reminder&quot; -ClearErrorVariable
}
[/code]

<a href="https://gist.github.com/6184059">https://gist.github.com/6184059</a>

For the memberof Group I recommend to use the SID instead of the DN. I'll show you how the get the SID:

[code lang="ps"]

PS C:Userssa-spadmin&gt; (Get-QADGroup &quot;SPO_PasswordNotification&quot;).Sid

  BinaryLength AccountDomainSid                                                               Value
  ------------ ----------------                                                               -----
			28 S-1-5-21-1744926098-708661255-2033415169                                       S-1-5-21-1744926098-708661255-2033415169-36648

PS C:Userssa-spadmin&gt; Get-QADUser -MemberOf &quot;S-1-5-21-1744926098-708661255-2033415169-36648&quot;

[/code]

<h1>Update</h1>

Part two: <a href="https://janikvonrotz.ch/2013/09/23/handling-user-password-change-and-expiration-issues-withoffice365-and-adfs-part-2/">https://janikvonrotz.ch/2013/09/23/handling-user-password-change-and-expiration-issues-withoffice365-and-adfs-part-2/</a>