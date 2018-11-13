---
title: ExchangeOnline Region Presettings
date: 2013-08-21T15:37:23+00:00
author: Janik von Rotz
slug: exchangeonline-region-presettings
images:
  - /wp-content/uploads/2013/08/exchange-2013-e1393417827333.jpg
categories:
  - Office 365
  - Scripting
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

<p style="text-align: center;">![Outlook Regional Settings](/wp-content/uploads/2013/08/Outlook-Regional-Settings.png)</p>

Despite it's not a lot of effort to change that, the timezone settings can be very confusing.

Running this script on random windows machine will open a ExchangeOnline PowerShell remote session and import them in your local session.

After the import it's the local session pretends to be live on the ExchangeOnline server, your able to run every PowerShell Exchange CMDlet. Yeapeeee :)

```powershell
try{

    #--------------------------------------------------#
    # settings
    #--------------------------------------------------#

    $Language = "de-CH"
    $TimeZone = "W. Europe Standard Time"
    $DateFormat = "dd.MM.yyyy"

    #--------------------------------------------------#
    # main
    #--------------------------------------------------#

    Import-Module ActiveDirectory

    $Credential = Import-PSCredential $(Get-ChildItem -Path $PSconfigs.Path -Filter "Office365.credentials.config.xml" -Recurse).FullName

    $s = New-PSSession -ConfigurationName Microsoft.Exchange `
    -ConnectionUri https://ps.outlook.com/powershell `
    -Credential $(Get-Credential -Credential $Credential) `
    -Authentication Basic `
    -AllowRedirection
    Import-PSSession $s

    $ADUsers = Get-ADUser -Filter {Mail -like "*"} -Properties sn, telephoneNumber, title
    $Mailboxes = $(Get-Mailbox)

    foreach($Mailbox in $Mailboxes){

        Write-Progress -Activity "Update settings" -status $($Mailbox.Name) -percentComplete ([Int32](([Array]::IndexOf($Mailboxes, $Mailbox)/($Mailboxes.count))*100))

        Write-Host "Set mailbox language settings for $($Mailbox.Name)"

        Set-MailboxRegionalConfiguration $Mailbox.Alias -Language $Language -TimeZone $TimeZone -LocalizeDefaultFolderName -DateFormat $DateFormat

        Write-Host "Set signature for $($Mailbox.Name)"

        $ADUser = $ADUsers | where{$_.UserPrincipalName -eq $Mailbox.UserPrincipalName} | select -First 1

        # get phone number
        if($ADuser.telephoneNumber -eq $null){
            $telephoneNumber = "+41 41 369 65 65"
        }else{
            $telephoneNumber = $ADuser.telephoneNumber
        }

        $Html = get-Content -Path $(Get-ChildItem -Path $PStemplates.Path -Filter "vbl signature.html" -Recurse).FullName
        $Text = get-Content -Path $(Get-ChildItem -Path $PStemplates.Path -Filter "vbl signature.txt" -Recurse).FullName

        $Html = $Html -replace "%%Firstname%%",$ADUser.givenname `
            -replace "%%Lastname%%",$Aduser.sn `
            -replace "%%Title%%",$Aduser.title `
            -replace "%%PhoneNumber%%",$telephoneNumber `
            -replace "%%FaxNumber%%","041 369 65 00"

        $Text = $Text -replace "%%Firstname%%",$ADUser.givenname `
            -replace "%%Lastname%%",$Aduser.sn `
            -replace "%%Title%%",$Aduser.title `
            -replace "%%PhoneNumber%%",$telephoneNumber `
            -replace "%%FaxNumber%%","041 369 65 00"

        Set-MailboxMessageConfiguration -Identity $Mailbox.Alias -SignatureHtml $HTML -AutoAddSignature $true -SignatureText $TEXT

    }

    # delete pssession
    Remove-PSSession $s

}catch{

    Send-PPErrorReport -FileName "DirSync.mail.config.xml" -ScriptName $MyInvocation.InvocationName

}
```

<a href="https://gist.github.com/6294947" target="_blank">https://gist.github.com/6294947</a>