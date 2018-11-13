---
title: Introducing new SharePoint Online Cmdlets
date: 2014-02-24T11:19:56+00:00
author: Janik von Rotz
slug: introducing-new-sharepoint-online-cmdlets
images:
  - /wp-content/uploads/2014/02/SharePoint-Online.jpg
categories:
  - Office 365
tags:
  - cmdlets
  - on-premise
  - powershell
  - sharepoint
---
<div id="head">

So far Micosoft doesn't offer the same amount of <a href="https://technet.microsoft.com/en-us/library/fp161388.aspx">cmdlets to manage a SharePoint Online</a> instance as they do for an on-premise installation.

As many other SharePoint administrators I can't wait to get new cmdlets. Luckily Jeffrey Paarhuis recently published a project on Microsoft Codeplex and showed how simple it is to manage a SharePoint installation just with client-side libraries and functions.

Seconds later I've migrated his functions into my project: <a href="https://github.com/janikvonrotz/PowerShell-PowerUp">https://github.com/janikvonrotz/PowerShell-PowerUp</a>

I'll introduce you the new:

<!--more-->
<h1><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/wiki/SharePoint-Online-Commands">SharePoint Online Commands</a></h1>
These functions are part of <a href="https://jeffreypaarhuis.com/">Jeffrey Paarhuis</a> project: <a href="https://sharepointpowershell.codeplex.com/">Client-side SharePoint PowerShell</a>. I've imported the functions of this project and updated the naming concept and metadata.

In order to use these commands you have to install the Managed .NET Client-Side Object Model (CSOM) of SharePoint 2013. Run the command `Install-PPApp "SharePoint Server 2013 Client Components SDK"` to install this app.
<ul>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOCalculatedFieldtoList.ps1">`Add-SPOCalculatedFieldtoList`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOChoiceFieldtoList.ps1">`Add-SPOChoiceFieldtoList`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOChoicesToField.ps1">`Add-SPOChoicesToField`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOCurrencyFieldtoList.ps1">`Add-SPOCurrencyFieldtoList`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPODateTimeFieldtoList.ps1">`Add-SPODateTimeFieldtoList`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPODocumentLibrary.ps1">`Add-SPODocumentLibrary`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOField.ps1">`Add-SPOField`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOFieldsToList.ps1">`Add-SPOFieldsToList`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOFolder.ps1">`Add-SPOFolder`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOGroup.ps1">`Add-SPOGroup`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOList.ps1">`Add-SPOList`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOListItems.ps1">`Add-SPOListItems`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPONoteFieldtoList.ps1">`Add-SPONoteFieldtoList`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPONumberFieldtoList.ps1">`Add-SPONumberFieldtoList`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOPrincipalToGroup.ps1">`Add-SPOPrincipalToGroup`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOTextFieldtoList.ps1">`Add-SPOTextFieldtoList`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOUserFieldtoList.ps1">`Add-SPOUserFieldtoList`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOWebpart.ps1">`Add-SPOWebpart`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Connect-SPO.ps1">`Connect-SPO`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Convert-SPOFileVariablesToValues.ps1">`Convert-SPOFileVariablesToValues`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Convert-SPOStringVariablesToValues.ps1">`Convert-SPOStringVariablesToValues`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Copy-SPOFile.ps1">`Copy-SPOFile`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Copy-SPOFolder.ps1">`Copy-SPOFolder`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Enable-SPOFeature.ps1">`Enable-SPOFeature`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Find-SPOFieldName.ps1">`Find-SPOFieldName`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Get-SPOGroup.ps1">`Get-SPOGroup`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Get-SPOPrincipal.ps1">`Get-SPOPrincipal`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Get-SPORole.ps1">`Get-SPORole`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Join-SPOParts.ps1">`Join-SPOParts`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Request-SPOYesOrNo.ps1">`Request-SPOYesOrNo`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Save-SPOFile.ps1">`Save-SPOFile`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Set-SPOCustomMasterPage.ps1">`Set-SPOCustomMasterPage`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Set-SPODocumentPermissions.ps1">`Set-SPODocumentPermissions`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Set-SPOListPermissions.ps1">`Set-SPOListPermissions`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Set-SPOMasterPage.ps1">`Set-SPOMasterPage`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Set-SPOWebPermissions.ps1">`Set-SPOWebPermissions`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Submit-SPOCheckIn.ps1">`Submit-SPOCheckIn`</a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Submit-SPOCheckOut.ps1">`Submit-SPOCheckOut`</a></li>
    <li>`<a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Test-SPOField.ps1">Test-SPOField</a>`</li>
</ul>
<h2>Example</h2>
[![SPO Example](/wp-content/uploads/2014/02/SPO-Example.jpg)](/wp-content/uploads/2014/02/SPO-Example.jpg)

</div>