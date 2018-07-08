---
title: 'For those who couldn’t attend SharePoint Conference 2014'
date: 2014-03-13T09:23:00+00:00
author: Janik von Rotz
slug: for-those-who-couldnt-attend-sharepoint-conference-2014
dsq_thread_id:
  - "2419566303"
image: /wp-content/uploads/2014/03/SharePoint-Conference-2014-e1394698189555.png
categories:
  - Business
  - Office 365
  - SharePoint
  - SharePoint Online
tags:
  - awesome
  - bill
  - clinton
  - conference
  - download
  - for
  - hangover
  - people
  - poor
  - sharepoint
  - slides
  - vegas
  - videos
---
For those who couldn't attend SharePoint Conference 2014 including me, the guy from <a href="https://absolute-sharepoint.com/">Absolute SharePoint Blog</a> has published a script to download all slides and videos of the presentations at SPC2014.
<!--more-->
By default the script does:

<ul>
<li>Downloads all the SPC14 Sessions and Slides</li>
<li>Groups them by folders</li>
<li>Makes sure no errors come up due to Illegal File names.</li>
<li>If you stop the script and restart in the middle, it will start where it left off and not from beginning.</li>
</ul>

Due to the immense file size of a bit under 70GB I've improved the script so that you can choose whether to include the videos or slides in the download or not.

Copy this script to `C:\spc14` and execute it.

```powershell
# settings 
$DownloadVideos = $false
$DownloadSlides = $true


[Environment]::CurrentDirectory=(Get-Location -PSProvider FileSystem).ProviderPath 
$rss = (new-object net.webclient)

# Grab the RSS feed for the MP4 downloads

# SharePoint Conference 2014 Videos
$a = ([[xml]]$rss.downloadstring("https://channel9.msdn.com/Events/SharePoint-Conference/2014/RSS/mp4high")) 
$b = ([[xml]]$rss.downloadstring("https://channel9.msdn.com/Events/SharePoint-Conference/2014/RSS/slides")) 

#other qualities for the videos only. Choose the one you want!
# $a = ([[xml]]$rss.downloadstring("https://channel9.msdn.com/Events/SharePoint-Conference/2014/RSS/mp4")) 
# $a = ([[xml]]$rss.downloadstring("https://channel9.msdn.com/Events/SharePoint-Conference/2014/RSS/mp3")) 

#Preferably enter something not too long to not have filename problems! cut and paste them afterwards
$downloadlocation = "C:\spc14"

if (-not (Test-Path $downloadlocation)) { 
	Write-Host "Folder $fpath dosen't exist. Creating it..."  
	New-Item $downloadlocation -type directory 
}
cd $downloadlocation



#Download all the slides	
if($DownloadSlides){$b.rss.channel.item | foreach{   

	$code = $_.comments.split("/") | select -last 1	   
	
	# Grab the URL for the PPTX file
	$urlpptx = New-Object System.Uri($_.enclosure.url)  
    $filepptx = $code + "-" + $_.creator + " - " + $_.title.Replace(":", "-").Replace("?", "").Replace("/", "-").Replace("<", "").Replace("|", "").Replace('"',"").Replace("*","")
	$filepptx = $filepptx.substring(0, [System.Math]::Min(120, $filepptx.Length))
	$filepptx = $filepptx + ".pptx" 
	
	if ($code -ne ""){
	
		 $folder = $code + " - " + $_.title.Replace(":", "-").Replace("?", "").Replace("/", "-").Replace("<", "").Replace("|", "").Replace('"',"").Replace("*","")
		 $folder = $folder.substring(0, [System.Math]::Min(100, $folder.Length))
	
	}else{
	
		$folder = "NoCodeSessions"
	}
	
	if (-not (Test-Path $folder)){ 
		Write-Host "Folder $folder dosen't exist. Creating it..."  
		New-Item $folder -type directory 
	}	

	# Make sure the PowerPoint file doesn't already exist
	if (!(test-path $folder\$filepptx)){ 	
	
		# Echo out the  file that's being downloaded
		$filepptx
		$wc = (New-Object System.Net.WebClient)  

		# Download the MP4 file
		$wc.DownloadFile($urlpptx, $filepptx)
		mv $filepptx $folder 

	}
}}

#download all the mp4

# Walk through each item in the feed 
if($DownloadVideos){$a.rss.channel.item | foreach{   
	
	$code = $_.comments.split("/") | select -last 1	   
	
	# Grab the URL for the MP4 file
	$url = New-Object System.Uri($_.enclosure.url)  
	
	# Create the local file name for the MP4 download
	$file = $code + "-" + $_.creator + "-" + $_.title.Replace(":", "-").Replace("?", "").Replace("/", "-").Replace("<", "").Replace("|", "").Replace('"',"").Replace("*","")
	$file = $file.substring(0, [System.Math]::Min(120, $file.Length))
	$file = $file + ".mp4"  
	
	if ($code -ne ""){
	
		 $folder = $code + " - " + $_.title.Replace(":", "-").Replace("?", "").Replace("/", "-").Replace("<", "").Replace("|", "").Replace('"',"").Replace("*","")
		 $folder = $folder.substring(0, [System.Math]::Min(100, $folder.Length))
	}else{
	
		$folder = "NoCodeSessions"
	}
	
	if (-not (Test-Path $folder)){ 
		Write-Host "Folder $folder) dosen't exist. Creating it..."  
		New-Item $folder -type directory 
	}
		
	# Make sure the MP4 file doesn't already exist

	if (!(test-path $folder\$file)){ 	
		# Echo out the  file that's being downloaded
		$file
		$wc = (New-Object System.Net.WebClient)  

		# Download the MP4 file
		$wc.DownloadFile($url, $file)
		mv $file $folder
	}
}}
```

The spc14 folder should look like this after downloading all the slides and videos.

[![SharePoint Conference 2014 Downloads](/wp-content/uploads/2014/03/SharePoint-Conference-2014-Downloads.png)](/wp-content/uploads/2014/03/SharePoint-Conference-2014-Downloads.png)

You'll find the original script on <a href="https://gallery.technet.microsoft.com/PowerShell-Script-to-all-04e92a63">Technet</a>