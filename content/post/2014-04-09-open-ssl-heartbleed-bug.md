---
title: Open SSL Heartbleed Bug
date: 2014-04-09T15:25:47+00:00
author: Janik Vonrotz
slug: open-ssl-heartbleed-bug
images:
  - /wp-content/uploads/2014/03/OpenSSL-Logo.png
categories:
  - Security
  - Web server
tags:
  - openssl
  - security
  - ssl
  - tls
---
For those who missed it. The OpenSSL project has recently announced a security vulnerability in OpenSSL affecting versions 1.0.1 and 1.0.2 (CVE-2014-0160).

Details of the bug are available here: [The Heartbleed Bug](http://heartbleed.com/)

You can check you website here: [Heartbleed test](http://filippo.io/Heartbleed/)

Details and update instructions from the websites of your Linux vendor of choice:
* [Amazon Linux AMI](https://aws.amazon.com/amazon-linux-ami/security-bulletins/ALAS-2014-320/)
* [Red Hat](https://rhn.redhat.com/errata/RHSA-2014-0376.html)
* [Ubuntu](http://www.ubuntu.com/usn/usn-2165-1/)

On Ubuntu the update is simply done by executing these commands:

	sudo apt-get update
	sudo apt-get upgrade

The following command shows (after an upgrade) all services that need to be restarted.

	ps uwwp $(sudo find /proc -maxdepth 2 -name maps -exec grep -HE '/libssl\.so.* \(deleted\)' {} \; | cut -d/ -f3 | sort -u)