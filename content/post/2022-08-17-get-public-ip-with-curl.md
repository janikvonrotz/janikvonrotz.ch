---
title: "Get public IP with Curl"
slug: get-public-ip-with-curl
date: 2022-08-17T08:39:54+02:00
categories:
 - System tooling
tags:
 - linux
 - curl
 - ipv4
images:
 - /wp-content/uploads/2013/08/Linux-Logo.jpg
---

Getting the public IP of your machine (running in the cloud) is simple. You probably find yourself on the command line and want to check if a DNS entry is resolved correctly. Login into the hoster's dashboard and checking the public IP there is too much of hassle.

<!--more-->

Simply run this command to get the public IP of your machine:

```console
curl ifconfig.me
```

Example output:

```console
janikvonrotz@cratos ➜  ~ curl ifconfig.me
78.47.134.58
```

The website <https://ifconfig.me/> simply returns the IP if queried with curl.

You can also run `curl ifconfig.me/all` to get more details.