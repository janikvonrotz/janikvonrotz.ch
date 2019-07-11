---
title: Apache vHost with SSL Certicate converted from .pfx Export file
date: 2014-01-27T14:03:40+00:00
author: Janik Vonrotz
slug: apache-vhost-with-ssl-certicate-converted-from-pfx-export-file
images:
  - /wp-content/uploads/2013/11/apache.jpg
categories:
  - Web server
tags:
  - apache
  - certificate
  - private key
  - web server
  - ssl
  - virtual host
---
This is an simple example for an Apache vHost SSL vHost configuration:

```
<VirtualHost 192.168.0.1:443>
DocumentRoot /var/www/
SSLEngine on
SSLCertificateFile /path/to/certificate.crt
SSLCertificateKeyFile /path/to/keyfile.key
</VirtualHost>
```

<!--more-->

When buying a certificate from you CA (Certification Authority) e.g. a wildcard certificate for *.domain.com, you have to convert this file to different formats in order to use them with you Apache installation.

To convert these files use <a href="https://www.openssl.org/" target="_blank">OpenSSL</a>.

First file you'll need is the public certificate.

```
openssl pkcs12 -in [yourfile.pfx] -clcerts -nokeys -out [certificate.crt]
```

Now you can chose between the encrypted and decrypted key file.

If chosing the encrypted key file, Apache will prompt every time starting or starting the web service for the certificate pass-phrase.

```
openssl pkcs12 -in [yourfile.pfx] -nocerts -out [keyfile-encrypted.key]
```

Otherwise Apache won't prompt for an pass-pharase, but be aware, if you're losing this decrypted key file you certificate will be worthless.

```
openssl rsa -in [keyfile-encrypted.key] -out [keyfile-decrypted.key]
```