---
title: 'My Dogecoin experience: mining server'
date: 2014-05-30T14:07:11+00:00
author: Janik von Rotz
slug: my-dogecoin-experience-mining-server    
images:
  - /wp-content/uploads/2014/05/Dogecoin-future.jpg
categories:
  - Blockchain
tags:
  - dogecoin
  - guide
  - mining
---
In my <a href="https://janikvonrotz.ch/2014/05/08/my-dogecoin-experience-part-1-mining-hardware/" title="last post">last post</a> I've written about the hardware components I've bought for my dogecoin mining server.
This time I show you the finished mining rig and summarize the most important configurations for the mining software.
<!--more-->
# My mining rig

<video width="1280" height="720" controls><source src="/wp-content/uploads/2014/05/My-Dogecoin-mining-server.mp4" type="video/mp4">Your browser does not support the video tag.</video>

# Basic Setup

This tutorial assumes you have the following setup:

* Windows 7 Service Pack 1
* 3 x Asus R9 270x graphics card

Download and install the latest version of:

[cgminer by ckolivas](https://github.com/ckolivas/cgminer)
[AMD Catalyst driver](http://support.amd.com/en-us/kb-articles/Pages/latest-catalyst-windows-beta.aspx)

In case to this cgminer won't work, try this one:

[cgminer by kalroth](https://github.com/Kalroth/cgminer-3.7.2-kalroth)

To monitor the cpu and ram usage of the server I'll use the built-in resource monitor for windows.

# Test Setup

After unziping the cgminer add a configuration file named `cgminer.conf` to the cgminer folder.

Add the following json configuration to config file, of course you have to replace the pool credentials.

```
{
	"pools" : [
		{
			"url" : "stratum+tcp://www.suchcoins.com:3333",
			"user" : "weblogin.WorkerName",
			"pass" : "workerPassword"
		}
	],
	"scrypt" : true
}
```

Next add a batch `cgminer.bat` to the same folder and add this commands:

```
setx GPU_MAX_ALLOC_PERCENT 100
setx GPU_USE_SYNC_OBJECTS 1
cgminer.exe
```

Finally double click the batch file. Cgminer should start to mine some coins.

# Productive Setup

Now we are going to tweak our cgminer installation. For this purpose we have to add a bunch of parameters to our batch file.

## cgminer Parameter

**Thread Concurrency**

--thread-concurrency
is a parameter to tell your graphics cards how many different tasks it should be juggling at the same time. Depending on your card, this can be set for anything between 2-30,000.  It may or may not help your performance, but most people set it between 10,000-25000.

**GPU Overclocking Parameters**

--gpu-memclock and --gpu-engine
Allows you to specifically overclock your cards. If you don’t know the defaults off the top of your head, a quick way is to go into cgminer and Press G, it will take you to the graphics card details screen.
E: would be your --gpu-engine and  M: --gpu-memclock for GPU 1.

**Intensity**

-I
A value between 1-20, this is a general setting that makes our graphics cards work harder. 
 Putting it on 20 will put your cards into high speed. Intensity is usually the last parameter that you will set, after you tweak everything else.

**Autofan and Temp Targets**

--auto-fan 
gives cgminer more control on keeping your cards a specific temperature. 

--temp-target 80
tells cgminer to keep your card around 80.  

--temp-overheat 85  
to help prevent overheating. 

**Work Size**

-w 
is the parameter for Work Size, or the size of jobs your cards will be doing.  Most people flag this at -256 to start for mid tier cards.  But the effect this parameter is described as “minimal”.

-g
This is labeled as an optional parameter that can give some small boost, but shouldn’t be be the first thing you go after.  It seems to crash cgminer if it is set any higher than 4, and many people leave it at 1 or 2.  Some people have gotten a lot more than others out of this.

## My configurations

This is my first configuration approach:
`-I 18 -g 1 -s 265 --thread-concurrency 20000 --auto-fan`
Now each GPU runs with about 400kH/s. It's not that much, I've seen others got about 450kH/s with the same hardware.

# Source

[http://linustechtips.com/main/topic/95726-r9-270x-485khs-setup](http://linustechtips.com/main/topic/95726-r9-270x-485khs-setup)
[https://litecoin.info/Mining_hardware_comparison](https://litecoin.info/Mining_hardware_comparison)
[http://www.minedogecoin.com/getting-such-dogecoin-a-basic-guide-to-gpu-mining-with-cgminer](http://www.minedogecoin.com/getting-such-dogecoin-a-basic-guide-to-gpu-mining-with-cgminer)