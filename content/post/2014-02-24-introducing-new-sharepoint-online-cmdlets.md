---
id: 1113
title: Introducing new SharePoint Online Cmdlets
date: 2014-02-24T11:19:56+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=1113
permalink: /2014/02/24/introducing-new-sharepoint-online-cmdlets/
dsq_thread_id:
  - "2312627481"
image: /wp-content/uploads/2014/02/SharePoint-Online.jpg
categories:
  - SharePoint Online
tags:
  - additional
  - cmdlets
  - new
  - on-premise
  - online
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

In order to use these commands you have to install the Managed .NET Client-Side Object Model (CSOM) of SharePoint 2013. Run the command <code>Install-PPApp "SharePoint Server 2013 Client Components SDK"</code> to install this app.
<ul>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOCalculatedFieldtoList.ps1"><code>Add-SPOCalculatedFieldtoList</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOChoiceFieldtoList.ps1"><code>Add-SPOChoiceFieldtoList</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOChoicesToField.ps1"><code>Add-SPOChoicesToField</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOCurrencyFieldtoList.ps1"><code>Add-SPOCurrencyFieldtoList</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPODateTimeFieldtoList.ps1"><code>Add-SPODateTimeFieldtoList</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPODocumentLibrary.ps1"><code>Add-SPODocumentLibrary</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOField.ps1"><code>Add-SPOField</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOFieldsToList.ps1"><code>Add-SPOFieldsToList</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOFolder.ps1"><code>Add-SPOFolder</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOGroup.ps1"><code>Add-SPOGroup</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOList.ps1"><code>Add-SPOList</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOListItems.ps1"><code>Add-SPOListItems</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPONoteFieldtoList.ps1"><code>Add-SPONoteFieldtoList</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPONumberFieldtoList.ps1"><code>Add-SPONumberFieldtoList</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOPrincipalToGroup.ps1"><code>Add-SPOPrincipalToGroup</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOTextFieldtoList.ps1"><code>Add-SPOTextFieldtoList</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOUserFieldtoList.ps1"><code>Add-SPOUserFieldtoList</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Add-SPOWebpart.ps1"><code>Add-SPOWebpart</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Connect-SPO.ps1"><code>Connect-SPO</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Convert-SPOFileVariablesToValues.ps1"><code>Convert-SPOFileVariablesToValues</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Convert-SPOStringVariablesToValues.ps1"><code>Convert-SPOStringVariablesToValues</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Copy-SPOFile.ps1"><code>Copy-SPOFile</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Copy-SPOFolder.ps1"><code>Copy-SPOFolder</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Enable-SPOFeature.ps1"><code>Enable-SPOFeature</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Find-SPOFieldName.ps1"><code>Find-SPOFieldName</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Get-SPOGroup.ps1"><code>Get-SPOGroup</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Get-SPOPrincipal.ps1"><code>Get-SPOPrincipal</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Get-SPORole.ps1"><code>Get-SPORole</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Join-SPOParts.ps1"><code>Join-SPOParts</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Request-SPOYesOrNo.ps1"><code>Request-SPOYesOrNo</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Save-SPOFile.ps1"><code>Save-SPOFile</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Set-SPOCustomMasterPage.ps1"><code>Set-SPOCustomMasterPage</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Set-SPODocumentPermissions.ps1"><code>Set-SPODocumentPermissions</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Set-SPOListPermissions.ps1"><code>Set-SPOListPermissions</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Set-SPOMasterPage.ps1"><code>Set-SPOMasterPage</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Set-SPOWebPermissions.ps1"><code>Set-SPOWebPermissions</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Submit-SPOCheckIn.ps1"><code>Submit-SPOCheckIn</code></a></li>
    <li><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Submit-SPOCheckOut.ps1"><code>Submit-SPOCheckOut</code></a></li>
    <li><code><a href="https://github.com/janikvonrotz/PowerShell-PowerUp/tree/master/functions/SharePoint%20Online/Test-SPOField.ps1">Test-SPOField</a></code></li>
</ul>
<h2>Example</h2>
<a href="https://janikvonrotz.ch/wp-content/uploads/2014/02/SPO-Example.jpg"><img class="aligncenter size-full wp-image-1123" alt="SPO Example" src="https://janikvonrotz.ch/wp-content/uploads/2014/02/SPO-Example.jpg" width="1186" height="593" /></a>

</div>