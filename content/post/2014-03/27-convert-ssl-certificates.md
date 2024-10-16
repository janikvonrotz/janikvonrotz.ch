---
title: Convert SSL certificates
date: 2014-03-27T14:01:50+00:00
author: Janik von Rotz
slug: convert-ssl-certificates
images:
  - /wp-content/uploads/2014/03/OpenSSL-Logo.png
categories:
  - Security
tags:
  - certificate
  - encryption
  - private key
  - openssl
  - pkcs12
  - rsa
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9413205](https://gist.github.com/9413205).*  

# Requirements
 
* [Get a free verified SSL certificate from StartSSL (optional)](https://janikvonrotz.ch/2014/03/26/get-a-free-verified-ssl-certificate-from-startssl/)

# Instructions

When buying a certificate from you CA (Certification Authority) e.g. a wildcard certificate for *.example.org, you have to convert this file to different formats in order to use them with your webserver installation.
<!--more-->
To convert these files use OpenSSL.

First file you’ll need is the public certificate.

    sudo openssl pkcs12 -in [yourfile.pfx] -clcerts -nokeys -out [certificate.crt]
    
Now you can chose between the encrypted and decrypted key file.

If chosing the encrypted key file your webserver will prompt every time starting the web service for the certificate pass-phrase.

    sudo openssl pkcs12 -in [yourfile.pfx] -nocerts -out [keyfile-encrypted.key]
    
Otherwise your webserver won’t prompt for an pass-pharase, but be aware, if you’re losing this decrypted key file you certificate will be worthless.

    sudo openssl pkcs12 -in [yourfile.pfx] -nodes -out [keyfile-decrypted.key]
    
## Certificate chain

During the SSL negotiation, a server provides its certificate along with the "intermediate" certificates that exist between it and the root. This allows clients to validate the server's certificate without going through a discovery processes that not all browsers support, and for those that do, without an additional performance penalty.

Download the CA server certificate on their website

    sudo sh -c "cat [certificate.crt] > [certificate.crt.ca.bundle]"
    sudo sh -c "cat [certificate.ca.crt] >> [certificate.crt.ca.bundle]"