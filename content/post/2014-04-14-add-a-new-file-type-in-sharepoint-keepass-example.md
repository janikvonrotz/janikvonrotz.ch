---
title: 'Add a new file type in SharePoint - KeePass example'
date: 2014-04-14T08:15:07+00:00
author: Janik Vonrotz
slug: add-a-new-file-type-in-sharepoint-keepass-example
images:
  - /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - keepass
  - sharepoint
---
In this post I'll show you how to enable a new file type in SharePoint.

The goal is simple, for the KeePass database file it should be possible to open them the same way as it works for office documents in SharePoint.

![KeePass on SharePoint](/wp-content/uploads/2014/04/KeePass-on-SharePoint.jpg)
<!--more-->
# Icon

The KeePass database file has the extension **.kdbx**. For this file type we are going to add an icon.

Open the folder **C:\Program Files\Common Files\microsoft shared\Web Server Extensions\15\TEMPLATE\IMAGES** on your SharePoint server.

Now add this KeePass icon image file to the IMAGES folder: ![KeePass_Icon](/wp-content/uploads/2014/04/KeePass_Icon.png)

To associate the icon image the file extension we need to add an entry to the DOCICON file.

Edit the file **C:\Program Files\Common Files\microsoft shared\Web Server Extensions\15\TEMPLATE\XML\DOCICON.XML** and add this line to the `<ByExtension>` node:

```xml
<Mapping Key="kdbx" Value="KeePass_Icon.png" EditText="Keepass Password Manager"/>
```

# Open file

To enable file opening for the database file with the KeePass program run this PowerShell script on the SharePoint server:

```powershell
# get all webapplications
$SPWebApps = Get-SPWebApplication

# set browser mime type
$mimeType = "application/vnd.keepass"
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
```

This script adds the KeePass mime type on every SharePoint web application.

Make sure that the SharePoint web application containing the KeePass database file has set the browser file handling permissive.

In case you want to change that with PowerShell run this script on your SharePoint server:

```powershell
# get all webapplications
$SPWebApps = Get-SPWebApplication

# set global file handling
$SPWebApps | foreach-object {
 if($_.BrowserFileHandling -ne "permissive" ){
      $_.BrowserFileHandling = "permissive" 
      $_.Update()
  }
}
```

Last thing we need to do is to restart the IIS webserver, run the command `iisreset`.
