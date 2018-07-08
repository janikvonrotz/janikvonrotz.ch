---
title: Deploy PowerShell ActiveDirectory Module without installing the remote server tools
date: 2015-09-09T16:23:17+00:00
author: Janik von Rotz
permalink: /2015/09/09/deploy-powershell-activedirectory-module-without-installing-the-remote-server-tools/
dsq_thread_id:
  - "4113355907"
image: /wp-content/uploads/2015/06/PowerShell-logo-e1433137513315.png
categories:
  - PowerShell
tags:
  - active
  - directory
  - import
  - install
  - module
  - powershell
---
Make use of the PowerShell ActiveDirectory module always required to install the [Remote Server Administration Tools](http://www.microsoft.com/en-us/download/details.aspx?id=7887).
That sucks! We want it as simple as executing a script. 
<!--more-->
Let's build a PowerShell script that adds the ActiveDirectory Powershell module on any computer.

For this setup we require one file and one folder (with its content).

The file is: `Microsoft.ActiveDirectory.Management.dll` (it's stored somwhere here: `C:\Windows\Microsoft.NET\assembly\GAC_64\Microsoft.ActiveDirectory.Management`)  
And the folder is: `C:\Windows\System32\WindowsPowerShell\v1.0\Modules\ActiveDirectory` 

Ironically you'll find these files on a computer where the Remote Server Administration Tools are installed.

Now create folder structure as below and place the files inside:

* .../yourfolder/
  * ActiveDirectory/
     * *Place the folder files here*
  * Microsoft.ActiveDirectory.Management.dll
  * Install-ActiveDirectoryModule.ps1

Then update the **Install-ActiveDirectoryModule.ps1** file with the content below:

```powershell
function Add-AssemblyToGlobalAssemblyCache{

    <#
    .SYNOPSIS 
	    Installing Assemblies to Global Assembly Cache (GAC)

    .DESCRIPTION 
	    This script is an alternative to the GACUTIL available in 
	    the .NET Framework SDK. It will put the specified assembly
	    in the GAC.

    .EXAMPLE
        Add-AssemblyToGlobalAssemblyCache -AssemblyName C:\Temp\MyWorkflow.dll
    
        This command will install the file MyWorkflow.dll from the C:\Temp directory in the GAC.

    .EXAMPLE
        Dir C:\MyWorkflowAssemblies | % {$_.Fullname} | Add-AssemblyToGlobalAssemblyCache
    
        You can also pass the assembly filenames through the pipeline making it easy
        to install several assemblies in one run. The command abobe  will install 
        all assemblies from the directory C:\MyWorkflowAssemblies, run this command

    .PARAMETER AssemblyName
	    Full path of the assembly file

    .PARAMETER PassThru
        If set, script will pass the filename given through the pipeline    

    .NOTES 
	    April 18, 2012 | Soren Granfeldt (soren@granfeldt.dk) 
		    - initial version

    .LINK 
        http://blog.goverco.com
    #>

    PARAM
    (
	    [Parameter(Mandatory=$true, ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$True)]
	    [ValidateNotNullOrEmpty()]
	    [string] $Name = "",

	    [switch]$PassThru
    )
 
	if ($null -eq ([AppDomain]::CurrentDomain.GetAssemblies() |? { $_.FullName -eq "System.EnterpriseServices, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" })){
		[System.Reflection.Assembly]::Load("System.EnterpriseServices, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a") | Out-Null
	}
	$PublishObject = New-Object System.EnterpriseServices.Internal.Publish
           
	foreach($Assembly in $Name){

        if ( -not (Test-Path $Assembly -type Leaf)){
            throw "The assembly '$Assembly' does not exist."
        }

        $LoadedAssembly = [System.Reflection.Assembly]::LoadFile($Assembly)

        if ($LoadedAssembly.GetName().GetPublicKey().Length -eq 0){
            throw "The assembly '$Assembly' must be strongly signed."
        }
          
        Write-Host "Installing: $Assembly"
        $PublishObject.GacInstall($Assembly)

        if($PassThru){$_}
    }
}

$moduledirectory = "C:\Windows\System32\WindowsPowerShell\v1.0\Modules\ActiveDirectory"
$basepath = Split-Path -parent $MyInvocation.MyCommand.Definition

if(-not (Test-Path -Path $moduledirectory)){
    New-Item -Path $moduledirectory -ItemType directory | Out-Null
}

Copy-Item -Path (Join-Path $basepath "\ActiveDirectory\*") -Destination $moduledirectory -Recurse -Force

Add-AssemblyToGlobalAssemblyCache -Name (Join-Path $basepath "Microsoft.ActiveDirectory.Management.dll")
```

Got it! When executing the **Install-ActiveDirectoryModule.ps1** on another computer it should install the PowerShell ActiveDirectory module.