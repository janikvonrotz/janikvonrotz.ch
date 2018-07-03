---
id: 733
title: E-Mail report of unchecked SharePoint files
date: 2013-11-14T16:05:42+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=733
permalink: /2013/11/14/e-mail-report-of-unchecked-sharepoint-files/
dsq_thread_id:
  - "1965705690"
image: /wp-content/uploads/2013/07/PowerShell.png
categories:
  - SharePoint
tags:
  - files
  - mail
  - powershell
  - powershellprofile
  - report
  - script
  - sharepoint
  - unchecked
---
Having unchecked files in the SharePoint is not a big deal as long as you aware of the those files.

Being unaware of which files are unchecked can really harm the information integrity of the SharePoint.

In one of my last posts I've reported the<a title="SharePoint File Reporting Done Right" href="https://janikvonrotz.ch/2013/10/10/sharepoint-file-reporting-done-right/" target="_blank"> SharePoint storage space used by files</a> and found about 1 GB of unchecked files by several people.

As they are in most cases unaware of those files or even don't understand what the file check-in/check-out is about, it's almost impossible to handle this files.

<!--more-->

To handle this issue I've written this simple script that reports all unchecked files of a SharePoint installation, writes and sends a nice report by mail to the owner of those files.

Here's a report example:

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/11/Example-Mail-Report-of-unchecked-SharePoint-files.png">![Example Mail Report of unchecked SharePoint files](https://janikvonrotz.ch/wp-content/uploads/2013/11/Example-Mail-Report-of-unchecked-SharePoint-files.png)</a>

and here's the script:

```ps

<#
$Metadata = @{
	Title = "Send Mail Report Of Unchecked SharePoint Files"
	Filename = "Send-MailReportOfUncheckedSharePointFiles.ps1"
	Description = ""
	Tags = ""
	Project = "powershell, script, sharepoint, report, unchecked, file"
	Author = "Janik von Rotz"
	AuthorContact = "https://janikvonrotz.ch"
	CreateDate = "2013-11-14"
	LastEditDate = "2013-11-14"
	Url = ""
	Version = "1.0.0"
	License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit https://creativecommons.org/licenses/by-sa/3.0/ch/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#>

#--------------------------------------------------#
# modules
#--------------------------------------------------#
Add-Type -AssemblyName System.Web
if ((Get-PSSnapin 'Microsoft.SharePoint.PowerShell' -ErrorAction SilentlyContinue) -eq $null){Add-PSSnapin 'Microsoft.SharePoint.PowerShell'}

#--------------------------------------------------#
# load config
#--------------------------------------------------#
$Mail = Get-PPConfiguration $PSconfigs.Mail.Filter | %{$_.Content.Mail | where{$_.Name -eq "Report Checked Out SharePoint Files"}} | select -first 1

#--------------------------------------------------#
# main
#--------------------------------------------------#
Get-SPWebApplication | Get-SPSite | Get-SPWeb -Limit All | %{
    $SPWeb = $_
    $_.lists | %{
        $_.CheckedOutFiles | %{
            $_ | select *, @{L="FileUrl";E={$SPWeb.Site.Url + "/" + $_.Url}}, @{L="SiteUrl";E={($SPWeb.Site.Url + "/" + $_.Url) -replace "[^/]+$",""}}
        }
    }
} | Group-Object CheckedOutByEmail | %{

    $Subject = "$($_.Count) nicht eingecheckte Dateien auf dem SharePoint"

    $Body = @"
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"  "https://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd"> <html xmlns="https://www.w3.org/1999/xhtml"> <head> <style>
    body{
        font-size: 11pt;
        font-family: Calibri
    }
    table {
        margin: 1em;
        border-collapse: collapse;
    }
    td, th {
        margin: 0.3em;
        border: 1px #ccc solid;
    }
</style></head><body>

    <p>Guten Tag $($_.Group[0].CheckedOutByName)</p>

    <p>Sie erhalten diese E-Mail, weil Sie im Besitz von nicht eingecheckten Dateien sind,</br>
    welche als letztes vor einer Woche oder fürher bearbeitet worden sind.</p>

    <p>Wir bitten Sie diese Dateien einzuchecken oder zu löschen.</p>

    <p>Untersützung zu diesem Thema erhalten Sie <a href="https://office.microsoft.com/de-ch/sharepoint-workspace-help/auschecken-und-einchecken-von-dokumenten-in-ein-dateitool-HA010356922.aspx">hier</a>.</p>

    <h2>Übersicht ihrer nicht eingecheckten Dateien</h2>

    $($_.group | select @{L="Name";E={$_.LeafName}}, @{L="FileUrl";E={"<a href='$($_.FileUrl)'>$($_.FileUrl)</a>"}}, @{L="SiteUrl";E={"<a href='$($_.SiteUrl)'>$($_.SiteUrl)</a>"}}, TimeLastModified, @{L="Size";E={Format-FileSize $_.Length}} | where{$_.TimeLastModified -lt $(Get-Date).AddDays(-7)} | ConvertTo-Html -Fragment)

    <p>ACHTUNG! Dieses E-Mail wurde von einem unbeaufsichtigtem Konto verschickt, Antworten an den Sender dieser E-Mail werden nicht bearbeitet.</p>

</body></html>
"@

    Write-PPEventLog -Message "Send Mail to: $($_.Name) with subject: $Subject" -Source "Send Mail Report Of Unchecked SharePoint Files" -WriteMessage
    Send-MailMessage -To $_.Name -From $Mail.FromAddress -Subject $Subject -Body ([System.Web.HttpUtility]::HtmlDecode($Body)) -SmtpServer $Mail.OutSmtpServer -BodyAsHtml -Priority High -Encoding ([System.Text.Encoding]::UTF8)
    Write-PPErrorEventLog -ScriptPath $MyInvocation.InvocationName -ClearErrorVariable
}
```

Latest version of this script: <a href="https://gist.github.com/7467398" target="_blank">https://gist.github.com/7467398</a>

Requirements to run this script:

<ul>
    <li><a href="https://github.com/janikvonrotz/PowerShell-Profile" target="_blank">PowerShell Profile</a></li>
</ul>