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

```powershell

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

```

And here are the manipulation functions:

<strong>Get-HostFileEntries</strong>

```powershell

function Get-HostFileEntries{

<#
.SYNOPSIS
    Get every entry from the hosts file.

.DESCRIPTION
	Extracts host entrys fromt the hosts file and shows them as a PowerShell object array.

.EXAMPLE
	PS C:> Get-HostFileEntries
#>

<#
    [CmdletBinding()]
	param(
		[Parameter(Mandatory=$true)]
		[String]
		$Name
	)
#>

    #--------------------------------------------------#
    # main
    #--------------------------------------------------#
    $HostFileContent = get-content "$env:windirSystem32driversetchosts"
    $Entries = @()
    foreach($Line in $HostFilecontent){
        if(!$Line.StartsWith("#") -and $Line -ne ""){

            $IP = ([regex]"(([2]([0-4][0-9]|[5][0-5])|[0-1]?[0-9]?[0-9])[.]){3}(([2]([0-4][0-9]|[5][0-5])|[0-1]?[0-9]?[0-9]))").match($Line).value
            $DNS = ($Line -replace $IP, "") -replace  's+',""

            $Entry = New-ObjectHostFileEntry -IP $IP -DNS $DNS
            $Entries += $Entry
        }
    }
    if($Entries -ne $Null){
        $Entries
    }else{
        throw "No entries found in host file"
    }
}
```

<strong>Add-HostFileEntry</strong>

```powershell

function Add-HostFileEntry{

<#
.SYNOPSIS
    Add an new entry to the hosts file

.DESCRIPTION
	Add an new entry the hosts file.

.PARAMETER  IP
	IP address

.PARAMETER  DNS
	DNS address

.EXAMPLE
	PS C:> Get-HostFileEntry -IP "192.168.50.4" -DNS "local.wordpress.dev"
#>

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
    $HostFile = "$env:windirSystem32driversetchosts"
    $LastLine = ($ContentFile | select -Last 1)

    if($IP -match [regex]"(([2]([0-4][0-9]|[5][0-5])|[0-1]?[0-9]?[0-9])[.]){3}(([2]([0-4][0-9]|[5][0-5])|[0-1]?[0-9]?[0-9]))"){

        Write-Host "Add entry to hosts file: "$(if($IP){$IP + " "}else{})$(if($DNS){$DNS})
        if($LastLine -ne "" -and  $LastLine -ne 's+' -and $LastLine -ne $Null){Add-Content -Path $HostFile -Value "" -Encoding "Ascii"}
        Add-Content -Path $HostFile -Value ($IP + "       " + $DNS) -Encoding "Ascii"
    }
}
```

<strong>Remove-HostFileEntry</strong>

```powershell

function Remove-HostFileEntry{

<#
.SYNOPSIS
    Add an new entry to the hosts file

.DESCRIPTION
	Add an new entry the hosts file.

.PARAMETER  IP
	IP address

.PARAMETER  DNS
	DNS address

.EXAMPLE
	PS C:> Get-HostFileEntry -IP "192.168.50.4" -DNS "local.wordpress.dev"
#>

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
    $HostFile = "$env:windirSystem32driversetchosts"
    $HostFileContent = get-content $HostFile
    $HostFileContentNew = @()
    $Modification = $false

    foreach($Line in $HostFilecontent){
        if($Line.StartsWith("#") -or $Line -eq ""){

            $HostFileContentNew += $Line
        }else{

            $HostIP = ([regex]"(([2]([0-4][0-9]|[5][0-5])|[0-1]?[0-9]?[0-9])[.]){3}(([2]([0-4][0-9]|[5][0-5])|[0-1]?[0-9]?[0-9]))").match($Line).value
            $HostDNS = ($Line -replace $HostIP, "") -replace 's+',""

            if($HostIP -eq $IP -or $HostDNS -eq $DNS){

                Write-Host "Remove host file entry: "$(if($IP){$IP + " "}else{})$(if($DNS){$DNS})
                $Modification = $true
            }else{
                $HostFileContentNew += $Line
            }
        }
    }

    if($Modification){

        Set-Content -Path $HostFile -Value $HostFileContentNew
    }else{
        throw "Couldn't find entry to remove in hosts file."
    }
}

```

These functions are part of my project <a href="https://github.com/janikvonrotz/Powershell-Profile">PowerShell Profile</a>.