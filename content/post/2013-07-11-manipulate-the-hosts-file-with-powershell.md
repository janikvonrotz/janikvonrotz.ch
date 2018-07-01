---
id: 288
title: Manipulate the hosts file with PowerShell
date: 2013-07-11T08:38:06+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=288
permalink: /2013/07/11/manipulate-the-hosts-file-with-powershell/
dsq_thread_id:
  - "1487179908"
image: /wp-content/uploads/2013/07/PowerShell.png
categories:
  - PowerShell
tags:
  - file
  - function
  - hosts
  - manipulation
  - powershell
---
Recently I had to add a host to the hosts file. What made my angry about this was the fact that I had to do it manually, if you've ever used a unix/linux machine this feels so wrong.

That's why I've created several PowerShell functions to make the manipulation of the hosts file easier.

<!--more-->

First there is the PowerShell object:

<strong>New-ObjectHostFileEntry</strong>

[code lang="ps"]

function New-ObjectHostFileEntry{
    param(
        [string]$IP,
        [string]$DNS
    )
    New-Object PSObject -Property @{
        IP = $IP
        DNS = $DNS
    }
}

[/code]

And here are the manipulation functions:

<strong>Get-HostFileEntries</strong>

[code lang="ps"]

function Get-HostFileEntries{

&lt;#
.SYNOPSIS
    Get every entry from the hosts file.

.DESCRIPTION
	Extracts host entrys fromt the hosts file and shows them as a PowerShell object array.

.EXAMPLE
	PS C:&gt; Get-HostFileEntries
#&gt;

&lt;#
    [CmdletBinding()]
	param(
		[Parameter(Mandatory=$true)]
		[String]
		$Name
	)
#&gt;

    #--------------------------------------------------#
    # main
    #--------------------------------------------------#
    $HostFileContent = get-content &quot;$env:windirSystem32driversetchosts&quot;
    $Entries = @()
    foreach($Line in $HostFilecontent){
        if(!$Line.StartsWith(&quot;#&quot;) -and $Line -ne &quot;&quot;){

            $IP = ([regex]&quot;(([2]([0-4][0-9]|[5][0-5])|[0-1]?[0-9]?[0-9])[.]){3}(([2]([0-4][0-9]|[5][0-5])|[0-1]?[0-9]?[0-9]))&quot;).match($Line).value
            $DNS = ($Line -replace $IP, &quot;&quot;) -replace  's+',&quot;&quot;

            $Entry = New-ObjectHostFileEntry -IP $IP -DNS $DNS
            $Entries += $Entry
        }
    }
    if($Entries -ne $Null){
        $Entries
    }else{
        throw &quot;No entries found in host file&quot;
    }
}
[/code]

<strong>Add-HostFileEntry</strong>

[code lang="ps"]

function Add-HostFileEntry{

&lt;#
.SYNOPSIS
    Add an new entry to the hosts file

.DESCRIPTION
	Add an new entry the hosts file.

.PARAMETER  IP
	IP address

.PARAMETER  DNS
	DNS address

.EXAMPLE
	PS C:&gt; Get-HostFileEntry -IP &quot;192.168.50.4&quot; -DNS &quot;local.wordpress.dev&quot;
#&gt;

    [CmdletBinding()]
	param(
		[Parameter(Mandatory=$true)]
		[String]
		$IP,

        [Parameter(Mandatory=$true)]
		[String]
		$DNS
	)

    #--------------------------------------------------#
    # main
    #--------------------------------------------------#
    $HostFile = &quot;$env:windirSystem32driversetchosts&quot;
    $LastLine = ($ContentFile | select -Last 1)

    if($IP -match [regex]&quot;(([2]([0-4][0-9]|[5][0-5])|[0-1]?[0-9]?[0-9])[.]){3}(([2]([0-4][0-9]|[5][0-5])|[0-1]?[0-9]?[0-9]))&quot;){

        Write-Host &quot;Add entry to hosts file: &quot;$(if($IP){$IP + &quot; &quot;}else{})$(if($DNS){$DNS})
        if($LastLine -ne &quot;&quot; -and  $LastLine -ne 's+' -and $LastLine -ne $Null){Add-Content -Path $HostFile -Value &quot;&quot; -Encoding &quot;Ascii&quot;}
        Add-Content -Path $HostFile -Value ($IP + &quot;       &quot; + $DNS) -Encoding &quot;Ascii&quot;
    }
}
[/code]

<strong>Remove-HostFileEntry</strong>

[code lang="ps"]

function Remove-HostFileEntry{

&lt;#
.SYNOPSIS
    Add an new entry to the hosts file

.DESCRIPTION
	Add an new entry the hosts file.

.PARAMETER  IP
	IP address

.PARAMETER  DNS
	DNS address

.EXAMPLE
	PS C:&gt; Get-HostFileEntry -IP &quot;192.168.50.4&quot; -DNS &quot;local.wordpress.dev&quot;
#&gt;

    [CmdletBinding()]
	param(
		[String]
		$IP,

		[String]
		$DNS
	)

    #--------------------------------------------------#
    # main
    #--------------------------------------------------#
    $HostFile = &quot;$env:windirSystem32driversetchosts&quot;
    $HostFileContent = get-content $HostFile
    $HostFileContentNew = @()
    $Modification = $false

    foreach($Line in $HostFilecontent){
        if($Line.StartsWith(&quot;#&quot;) -or $Line -eq &quot;&quot;){

            $HostFileContentNew += $Line
        }else{

            $HostIP = ([regex]&quot;(([2]([0-4][0-9]|[5][0-5])|[0-1]?[0-9]?[0-9])[.]){3}(([2]([0-4][0-9]|[5][0-5])|[0-1]?[0-9]?[0-9]))&quot;).match($Line).value
            $HostDNS = ($Line -replace $HostIP, &quot;&quot;) -replace 's+',&quot;&quot;

            if($HostIP -eq $IP -or $HostDNS -eq $DNS){

                Write-Host &quot;Remove host file entry: &quot;$(if($IP){$IP + &quot; &quot;}else{})$(if($DNS){$DNS})
                $Modification = $true
            }else{
                $HostFileContentNew += $Line
            }
        }
    }

    if($Modification){

        Set-Content -Path $HostFile -Value $HostFileContentNew
    }else{
        throw &quot;Couldn't find entry to remove in hosts file.&quot;
    }
}

[/code]

These functions are part of my project <a href="https://github.com/janikvonrotz/Powershell-Profile">PowerShell Profile</a>.