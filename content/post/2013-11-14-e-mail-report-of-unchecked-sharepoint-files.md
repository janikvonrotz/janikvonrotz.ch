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

<a href="https://janikvonrotz.ch/wp-content/uploads/2013/11/Example-Mail-Report-of-unchecked-SharePoint-files.png"><img class="aligncenter size-full wp-image-734" alt="Example Mail Report of unchecked SharePoint files" src="https://janikvonrotz.ch/wp-content/uploads/2013/11/Example-Mail-Report-of-unchecked-SharePoint-files.png" width="1327" height="466" /></a>

and here's the script:

[code lang="ps"]

&lt;#
$Metadata = @{
	Title = &quot;Send Mail Report Of Unchecked SharePoint Files&quot;
	Filename = &quot;Send-MailReportOfUncheckedSharePointFiles.ps1&quot;
	Description = &quot;&quot;
	Tags = &quot;&quot;
	Project = &quot;powershell, script, sharepoint, report, unchecked, file&quot;
	Author = &quot;Janik von Rotz&quot;
	AuthorContact = &quot;https://janikvonrotz.ch&quot;
	CreateDate = &quot;2013-11-14&quot;
	LastEditDate = &quot;2013-11-14&quot;
	Url = &quot;&quot;
	Version = &quot;1.0.0&quot;
	License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit https://creativecommons.org/licenses/by-sa/3.0/ch/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#&gt;

#--------------------------------------------------#
# modules
#--------------------------------------------------#
Add-Type -AssemblyName System.Web
if ((Get-PSSnapin 'Microsoft.SharePoint.PowerShell' -ErrorAction SilentlyContinue) -eq $null){Add-PSSnapin 'Microsoft.SharePoint.PowerShell'}

#--------------------------------------------------#
# load config
#--------------------------------------------------#
$Mail = Get-PPConfiguration $PSconfigs.Mail.Filter | %{$_.Content.Mail | where{$_.Name -eq &quot;Report Checked Out SharePoint Files&quot;}} | select -first 1

#--------------------------------------------------#
# main
#--------------------------------------------------#
Get-SPWebApplication | Get-SPSite | Get-SPWeb -Limit All | %{
    $SPWeb = $_
    $_.lists | %{
        $_.CheckedOutFiles | %{
            $_ | select *, @{L=&quot;FileUrl&quot;;E={$SPWeb.Site.Url + &quot;/&quot; + $_.Url}}, @{L=&quot;SiteUrl&quot;;E={($SPWeb.Site.Url + &quot;/&quot; + $_.Url) -replace &quot;[^/]+$&quot;,&quot;&quot;}}
        }
    }
} | Group-Object CheckedOutByEmail | %{

    $Subject = &quot;$($_.Count) nicht eingecheckte Dateien auf dem SharePoint&quot;

    $Body = @&quot;
&lt;!DOCTYPE html PUBLIC &quot;-//W3C//DTD XHTML 1.0 Strict//EN&quot;  &quot;https://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd&quot;&gt; &lt;html xmlns=&quot;https://www.w3.org/1999/xhtml&quot;&gt; &lt;head&gt; &lt;style&gt;
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
&lt;/style&gt;&lt;/head&gt;&lt;body&gt;

    &lt;p&gt;Guten Tag $($_.Group[0].CheckedOutByName)&lt;/p&gt;

    &lt;p&gt;Sie erhalten diese E-Mail, weil Sie im Besitz von nicht eingecheckten Dateien sind,&lt;/br&gt;
    welche als letztes vor einer Woche oder fürher bearbeitet worden sind.&lt;/p&gt;

    &lt;p&gt;Wir bitten Sie diese Dateien einzuchecken oder zu löschen.&lt;/p&gt;

    &lt;p&gt;Untersützung zu diesem Thema erhalten Sie &lt;a href=&quot;https://office.microsoft.com/de-ch/sharepoint-workspace-help/auschecken-und-einchecken-von-dokumenten-in-ein-dateitool-HA010356922.aspx&quot;&gt;hier&lt;/a&gt;.&lt;/p&gt;

    &lt;h2&gt;Übersicht ihrer nicht eingecheckten Dateien&lt;/h2&gt;

    $($_.group | select @{L=&quot;Name&quot;;E={$_.LeafName}}, @{L=&quot;FileUrl&quot;;E={&quot;&lt;a href='$($_.FileUrl)'&gt;$($_.FileUrl)&lt;/a&gt;&quot;}}, @{L=&quot;SiteUrl&quot;;E={&quot;&lt;a href='$($_.SiteUrl)'&gt;$($_.SiteUrl)&lt;/a&gt;&quot;}}, TimeLastModified, @{L=&quot;Size&quot;;E={Format-FileSize $_.Length}} | where{$_.TimeLastModified -lt $(Get-Date).AddDays(-7)} | ConvertTo-Html -Fragment)

    &lt;p&gt;ACHTUNG! Dieses E-Mail wurde von einem unbeaufsichtigtem Konto verschickt, Antworten an den Sender dieser E-Mail werden nicht bearbeitet.&lt;/p&gt;

&lt;/body&gt;&lt;/html&gt;
&quot;@

    Write-PPEventLog -Message &quot;Send Mail to: $($_.Name) with subject: $Subject&quot; -Source &quot;Send Mail Report Of Unchecked SharePoint Files&quot; -WriteMessage
    Send-MailMessage -To $_.Name -From $Mail.FromAddress -Subject $Subject -Body ([System.Web.HttpUtility]::HtmlDecode($Body)) -SmtpServer $Mail.OutSmtpServer -BodyAsHtml -Priority High -Encoding ([System.Text.Encoding]::UTF8)
    Write-PPErrorEventLog -ScriptPath $MyInvocation.InvocationName -ClearErrorVariable
}
[/code]

Latest version of this script: <a href="https://gist.github.com/7467398" target="_blank">https://gist.github.com/7467398</a>

Requirements to run this script:

<ul>
    <li><a href="https://github.com/janikvonrotz/PowerShell-Profile" target="_blank">PowerShell Profile</a></li>
</ul>