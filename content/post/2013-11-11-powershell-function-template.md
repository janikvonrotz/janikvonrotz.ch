---
id: 694
title: PowerShell Function Template
date: 2013-11-11T16:04:39+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=694
permalink: /2013/11/11/powershell-function-template/
dsq_thread_id:
  - "1956333799"
image: /wp-content/uploads/2013/07/PowerShell.png
categories:
  - PowerShell
tags:
  - function
  - powershell
  - script
  - template
---
A good way to start writing a custom function in PowerShell is an advanced template like this.
This is my custom PowerShell function template, whenever I'm writing a new script I'll start with one of my templates. Having consistency in structure and naming of code is an important part in collaboration.
<!--more-->
You can use this template for whatever like and do you can do everything you'll like (except selling) with it.

```ps
<#
$Metadata = @{
	Title = ""
	Filename = ""
	Description = ""
	Tags = ""
	Project = ""
	Author = "Janik von Rotz"
	AuthorContact = "https://janikvonrotz.ch"
	CreateDate = "yyyyy-mm-dd hh:mm"
	LastEditDate = "yyyyy-mm-dd hh:mm"
	Url = ""
	Version = "0.0.0"
	License = @'
This work is licensed under the Creative Commons Attribution-ShareAlike 3.0 Switzerland License.
To view a copy of this license, visit https://creativecommons.org/licenses/by-sa/3.0/ch/ or
send a letter to Creative Commons, 444 Castro Street, Suite 900, Mountain View, California, 94041, USA.
'@
}
#>

function Test-AdvancedFunction{

<#
.SYNOPSIS
    A brief description of the function.

.DESCRIPTION
	A detailed description of the function.

.PARAMETER  ParameterA
	The description of the ParameterA parameter.

.PARAMETER  ParameterB
	The description of the ParameterB parameter.

.EXAMPLE
	PS C:> Get-Something -ParameterA 'One value' -ParameterB 32

.EXAMPLE
	PS C:> Get-Something 'One value' 32

.INPUTS
	System.String,System.Int32

.OUTPUTS
	System.String

.NOTES
	Additional information about the function go here.

.LINK
	about_functions_advanced

.LINK
	about_comment_based_help

#>

	[CmdletBinding()]
	param(

        # parameter options
        # validation
        # cast
        # name and default value

		[Parameter(Position=0, Mandatory=$true)]
		[ValidateNotNullOrEmpty()]
		[System.String]
		$Name,

		[Parameter(Position=1)]
		[ValidateNotNull()]
		[System.Int32]
		$Index

	)# param end

  #--------------------------------------------------#
  # settings
  #--------------------------------------------------#

  #--------------------------------------------------#
  # functions
  #--------------------------------------------------#

  #--------------------------------------------------#
  # modules
  #--------------------------------------------------#

  #--------------------------------------------------#
  # main
  #--------------------------------------------------#

    # This block is used to provide optional one-time pre-processing for the function.
    begin{

        Do-Something
    }# begin end

    # This block is used to provide record-by-record processing for the function.
    process{

    	try{

    	}# try end

    	catch{

    		throw

        }# catch end

        finally{

        }# finally end
    }# process end

    # This block is used to provide optional one-time post-processing for the function.
    end{

    }# end end
}# function end

```

Hope I'm going save you time in programming great things.

Check out the latest version of this template here: <a href="https://gist.github.com/5749458">https://gist.github.com/5749458</a>