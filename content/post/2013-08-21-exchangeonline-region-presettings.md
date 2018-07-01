---
id: 443
title: ExchangeOnline Region Presettings
date: 2013-08-21T15:37:23+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=443
permalink: /2013/08/21/exchangeonline-region-presettings/
dsq_thread_id:
  - "1624176205"
image: /wp-content/uploads/2013/08/exchange-2013-e1393417827333.jpg
categories:
  - Exchange
  - Office 365
  - PowerShell
tags:
  - exchange
  - office365
  - regional
  - script
  - settings
---
This post is part of my Office365 experience. Today I've made a script to get startet with the ExchangeOnline PowerShell console.

Every time a user logged in for the first time into the Office365 Outlook webapp, he had to set his regional configuration.

<!--more-->

<p style="text-align: center;"><img class="size-full wp-image-448 aligncenter" alt="Outlook Regional Settings" src="https://janikvonrotz.ch/wp-content/uploads/2013/08/Outlook-Regional-Settings.png" width="637" height="449" /></p>

Despite it's not a lot of effort to change that, the timezone settings can be very confusing.

Running this script on random windows machine will open a ExchangeOnline PowerShell remote session and import them in your local session.

After the import it's the local session pretends to be live on the ExchangeOnline server, your able to run every PowerShell Exchange CMDlet. Yeapeeee :)

[code lang="ps"]
&lt;pre&gt;try{

    #--------------------------------------------------#
    # settings
    #--------------------------------------------------#

    $Language = &quot;de-CH&quot;
    $TimeZone = &quot;W. Europe Standard Time&quot;
    $DateFormat = &quot;dd.MM.yyyy&quot;

    #--------------------------------------------------#
    # main
    #--------------------------------------------------#

    Import-Module ActiveDirectory

    $Credential = Import-PSCredential $(Get-ChildItem -Path $PSconfigs.Path -Filter &quot;Office365.credentials.config.xml&quot; -Recurse).FullName

    $s = New-PSSession -ConfigurationName Microsoft.Exchange `
    -ConnectionUri https://ps.outlook.com/powershell `
    -Credential $(Get-Credential -Credential $Credential) `
    -Authentication Basic `
    -AllowRedirection
    Import-PSSession $s

    $ADUsers = Get-ADUser -Filter {Mail -like &quot;*&quot;} -Properties sn, telephoneNumber, title
    $Mailboxes = $(Get-Mailbox)

    foreach($Mailbox in $Mailboxes){

        Write-Progress -Activity &quot;Update settings&quot; -status $($Mailbox.Name) -percentComplete ([Int32](([Array]::IndexOf($Mailboxes, $Mailbox)/($Mailboxes.count))*100))

        Write-Host &quot;Set mailbox language settings for $($Mailbox.Name)&quot;

        Set-MailboxRegionalConfiguration $Mailbox.Alias -Language $Language -TimeZone $TimeZone -LocalizeDefaultFolderName -DateFormat $DateFormat

        Write-Host &quot;Set signature for $($Mailbox.Name)&quot;

        $ADUser = $ADUsers | where{$_.UserPrincipalName -eq $Mailbox.UserPrincipalName} | select -First 1

        # get phone number
        if($ADuser.telephoneNumber -eq $null){
            $telephoneNumber = &quot;+41 41 369 65 65&quot;
        }else{
            $telephoneNumber = $ADuser.telephoneNumber
        }

        $Html = get-Content -Path $(Get-ChildItem -Path $PStemplates.Path -Filter &quot;vbl signature.html&quot; -Recurse).FullName
        $Text = get-Content -Path $(Get-ChildItem -Path $PStemplates.Path -Filter &quot;vbl signature.txt&quot; -Recurse).FullName

        $Html = $Html -replace &quot;%%Firstname%%&quot;,$ADUser.givenname `
            -replace &quot;%%Lastname%%&quot;,$Aduser.sn `
            -replace &quot;%%Title%%&quot;,$Aduser.title `
            -replace &quot;%%PhoneNumber%%&quot;,$telephoneNumber `
            -replace &quot;%%FaxNumber%%&quot;,&quot;041 369 65 00&quot;

        $Text = $Text -replace &quot;%%Firstname%%&quot;,$ADUser.givenname `
            -replace &quot;%%Lastname%%&quot;,$Aduser.sn `
            -replace &quot;%%Title%%&quot;,$Aduser.title `
            -replace &quot;%%PhoneNumber%%&quot;,$telephoneNumber `
            -replace &quot;%%FaxNumber%%&quot;,&quot;041 369 65 00&quot;

        Set-MailboxMessageConfiguration -Identity $Mailbox.Alias -SignatureHtml $HTML -AutoAddSignature $true -SignatureText $TEXT

    }

    # delete pssession
    Remove-PSSession $s

}catch{

    Send-PPErrorReport -FileName &quot;DirSync.mail.config.xml&quot; -ScriptName $MyInvocation.InvocationName

}&lt;/pre&gt;
[/code]

<a href="https://gist.github.com/6294947" target="_blank">https://gist.github.com/6294947</a>