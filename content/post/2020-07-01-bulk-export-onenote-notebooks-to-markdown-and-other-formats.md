---
title: "Bulk export OneNote notebooks to markdown and other formats"
slug: 01-bulk-export-onenote-notebooks-to-markdown-and-other-formats
date: 2020-07-01T20:17:34+02:00
categories:
 - Scripting
tags:
 - markdown
 - open source
images:
 - /images/unchained.jpg
---

I have decided to ditch corporate software and replace everything with open source software. It is an ongoing process that takes some time. Open source alternatives took some strides in recent years. One of the rising stars is [Nextcloud](https://nextcloud.com/). It is a self-hosted data platform that lets you keep control. Featurewise we do not have to start a discussion. Everything runs in your browser, there is a mobile app and client for all desktops.
<!--more-->

So in short I ditching OneDrive, Office 365 and move to Nextcloud and LibreOffice. As mentioned there are a lot of constraints. One of them is OneNote. Microsoft has integrated OneNote deeply into their Office ecosystem. The OneNote file format is proprietary and therefore not supported by other note taking app. Moreover OneDrive does not store the Notebooks as files, but simply as immutable links. Getting the data out of OneNote requires you to sync all notebooks with the desktop clients. Below I will show you a PowerShell script that I created to export all my notebooks into different formats.

Let my elaborate which formats I've chosen and why.

* Onepkg is used to archive the notebooks.
* Html preserves drawings best and does not require a client.
* Docx is used to convert the file into markdown.

Here is the script:

```powershell
# settings
$ExportPath = "C:\Users\$env:USERNAME\OneDrive\OneNoteExport\"

# Create OneNote application
$OneNote = New-Object -ComObject OneNote.Application

# Set note hierarchy
[xml]$Hierarchy = ""
$OneNote.GetHierarchy("", [Microsoft.Office.InterOp.OneNote.HierarchyScope]::hsPages, [ref]$Hierarchy)

# Get info foreach notebook
$Hierarchy.Notebooks.Notebook | ?{$_.name -eq 'TODO'} | %{
    $Notebook = $_
    $Name = $Notebook.name
    $NotebookPath = Join-Path -Path $ExportPath -ChildPath $Name

    Write-Host "Export Notebook: $Name"
    New-Item -Force -Path $NotebookPath -ItemType directory | Out-Null

    $NotebookOnepkgPath = Join-Path -Path $ExportPath -ChildPath "$($Name).onepkg"
    if (!(Test-Path -Path $NotebookOnepkgPath)) {
        Write-Host "Export Notebook as Onepkg: $NotebookOnepkgPath"
        $OneNote.Publish($Notebook.ID, $NotebookOnepkgPath, 1, "")
    }

    # Get info about each section
    $Notebook.ChildNodes| ?{$_.isRecycleBin -ne 'true'} | %{
        $Section = $_
        $SectionIndex = [array]::indexof($Notebook.ChildNodes, $_)
        $SectionName = "$($SectionIndex)_$($Section.name)"
        $SectionPath = Join-Path -Path $NotebookPath -ChildPath $SectionName

        Write-Host "Processing Section: $SectionName"
        New-Item -Force -Path $SectionPath -ItemType directory | Out-Null

        # Process pages
        $Section.ChildNodes | %{
            $Page = $_
            $PageIndex = [array]::indexof($Section.ChildNodes, $_)
            $PageName = "$($PageIndex)_$($Page.name)".Split([IO.Path]::GetInvalidFileNameChars()) -join '_'
            $PagePath = Join-Path -Path $SectionPath -ChildPath $PageName

            Write-Host "Processing Page: $PageName"
            New-Item -Force -Path $PagePath -ItemType directory | Out-Null

            $PageHtmPath = Join-Path -Path $PagePath -ChildPath 'index.htm'
            if (!(Test-Path -Path $PageHtmPath)) {
                Write-Host "Export Page as Htm: $PageHtmPath"
                $OneNote.Publish($Page.ID, $PageHtmPath, 7, "")
            }

            $PageDocxPath = Join-Path -Path $PagePath -ChildPath 'index.docx'
            if (!(Test-Path -Path $PageDocxPath)) {
                Write-Host "Export Page as Docx: $PageDocxPath"
                $OneNote.Publish($Page.ID, $PageDocxPath, 5, "")
            }

            $PageMdPath = Join-Path -Path $PagePath -ChildPath 'index.md'
            if (!(Test-Path -Path $PageMdPath)) {
                Write-Host "Convert Docx to Md: $PageMdPath"
                Set-Location $PagePath
                pandoc.exe --extract-media=./ .\index.docx -o index.md -t gfm
            }

            # Export Attachments
            $xml = ''
            $schema = @{one=”http://schemas.microsoft.com/office/onenote/2013/onenote”}
            $onenote.GetPageContent($Page.ID, [ref]$xml)
            $xml | Select-Xml -XPath "//one:Page/one:Outline/one:OEChildren/one:OE/one:InsertedFile" -Namespace $schema | %{
                $AttachmentPath = Join-Path -Path $PagePath -ChildPath $_.Node.preferredName
                Write-Host "Export Attachment: $($AttachmentPath)"
                Copy-Item -Force $_.Node.pathCache -Destination $AttachmentPath
            }
        }
    }
}
```

The OneNote desktop client must be started in order to run the script. Converting `docx` to `markdown` is done using [Pandoc](https://pandoc.org/). I had help from this [blog post](https://www.passbe.com/2019/08/01/bulk-export-onenote-2013-2016-pages-as-html/) and this [code snippet](https://gist.github.com/heardk/ded40b72056cee33abb18f3724e0a580).

Let me know if you have similar plans and if the script worked for you.