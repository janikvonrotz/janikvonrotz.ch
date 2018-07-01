---
id: 1930
title: 'Add a new file type in SharePoint &#8211; KeePass example'
date: 2014-04-14T08:15:07+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=1930
permalink: /2014/04/14/add-a-new-file-type-in-sharepoint-keepass-example/
dsq_thread_id:
  - "2610061515"
image: /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - KeePass
  - SharePoint
tags:
  - editing
  - enable
  - file
  - new
  - sharepoint
  - support
  - type
---
In this post I'll show you how to enable a new file type in SharePoint.

The goal is simple, for the KeePass database file it should be possible to open them the same way as it works for office documents in SharePoint.

<img src="https://janikvonrotz.ch/wp-content/uploads/2014/04/KeePass-on-SharePoint.jpg" alt="KeePass on SharePoint" width="755" height="405" class="aligncenter size-full wp-image-1934" />
<!--more-->
# Icon

The KeePass database file has the extension **.kdbx**. For this file type we are going to add an icon.

Open the folder **C:\Program Files\Common Files\microsoft shared\Web Server Extensions\15\TEMPLATE\IMAGES** on your SharePoint server.

Now add this KeePass icon image file to the IMAGES folder: <img src="https://janikvonrotz.ch/wp-content/uploads/2014/04/KeePass_Icon.png" alt="KeePass_Icon" width="32" height="32" class="aligncenter size-full wp-image-1931" />

To associate the icon image the file extension we need to add an entry to the DOCICON file.

Edit the file **C:\Program Files\Common Files\microsoft shared\Web Server Extensions\15\TEMPLATE\XML\DOCICON.XML** and add this line to the `<ByExtension>` node:

[code lang="xml"]
&lt;Mapping Key=&quot;kdbx&quot; Value=&quot;KeePass_Icon.png&quot; EditText=&quot;Keepass Password Manager&quot;/&gt;
[/code]

# Open file

To enable file opening for the database file with the KeePass program run this PowerShell script on the SharePoint server:

[code lang="ps"]
# get all webapplications
$SPWebApps = Get-SPWebApplication

# set browser mime type
$mimeType = &quot;application/vnd.keepass&quot;
$SPWebApps | foreach-object { 

    # If the MIME Type is not already on the allowed list for the Web Application 
    if(!$_.AllowedInlineDownloadedMimeTypes.Contains($mimeType)){ 

        # Add the MIME type to the allowed list and update the Web Application 
        $_.AllowedInlineDownloadedMimeTypes.Add($mimeType) 
        $_.Update() 
        Write-Host Added $mimeType to the allowed list for Web Application $_.Name 

    }else{ 

        # The MIME type was already allowed - can't add. Inform user 
        Write-Host Skipped Web Application $_.Name - $mimeType was already allowed 
    } 
}
[/code]

This script adds the KeePass mime type on every SharePoint web application.

Make sure that the SharePoint web application containing the KeePass database file has set the browser file handling permissive.

In case you want to change that with PowerShell run this script on your SharePoint server:

[code lang="ps"]
# get all webapplications
$SPWebApps = Get-SPWebApplication

# set global file handling
$SPWebApps | foreach-object {
 if($_.BrowserFileHandling -ne &quot;permissive&quot; ){
      $_.BrowserFileHandling = &quot;permissive&quot; 
      $_.Update()
  }
}
[/code]

Last thing we need to do is to restart the IIS webserver, run the command `iisreset`.
