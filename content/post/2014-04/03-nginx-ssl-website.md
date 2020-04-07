---
title: Nginx SSL website
date: 2014-04-03T07:54:04+00:00
author: Janik Vonrotz
slug: nginx-ssl-website
images:
  - /wp-content/uploads/2014/03/Nginx-Logo-e1394033855329.png
categories:
  - Security
tags:
  - certificate
  - compliance
  - private key
  - nginx
  - openssl
  - security
  - web server
  - ssl
---
*This post is part of my [Your own Virtual Private Server hosting solution](https://janikvonrotz.ch/your-own-virtual-private-server-hosting-solution/) project.*  
*Get the latest version of this article here: [https://gist.github.com/9408793](https://gist.github.com/9408793).*

# Introduction

This best practice shows you the most advanced SSL configurations for your Nginx website.
For productive usage it's recommended to use only public-signed certificates.
<!--more-->
# Requirements

* [Ubuntu server](https://janikvonrotz.ch/2014/03/13/deploy-ubuntu-server/)
* [Get a free verified SSL certificate from StartSSL (optional)](https://janikvonrotz.ch/2014/03/26/get-a-free-verified-ssl-certificate-from-startssl/)
* [Converted SSL certificates (optional)](https://janikvonrotz.ch/2014/03/27/convert-ssl-certificates/)
* [Nginx](https://janikvonrotz.ch/2014/03/31/install-nginx/)
* [Nginx minimal website](https://janikvonrotz.ch/2014/04/01/nginx-minimal-website/)

# Installation

Create a ssl folder to store key and cert files

    sudo mkdir /etc/nginx/ssl
    
Upload your key and cert files into this folder.

Now we need to generate stronger DHE parameter:

    cd /etc/ssl/certs
    sudo openssl dhparam -out dhparam.pem 4096

Add this Nginx configuration to your website config.

```
server{

        # set ssl port
        listen 443;
        
        ...
        
        # basic ssl configuration
        ssl on;
        ssl_certificate /etc/nginx/ssl/[certificate.crt.ca.bundle];
        ssl_certificate_key /etc/nginx/ssl/[host].key;

        # Force to use stronger DHE parameters 
        ssl_dhparam /etc/ssl/certs/dhparam.pem;
        
        # limitation of ssl protocols and algortyhtms
        
        # we don't want to support SSL v2 and SSL v3, it's known to be insecure
        # FIPS 140-2 compliance, TLS1+ only
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        
        # don't let the client decide what ciphers to use, we've told the server which to allow
        ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
        ssl_prefer_server_ciphers on;
        
        # reduce ssl cpu load
        
        # we want to enable ssl session resumption to avoid
        # having to start the handshake from scratch each page load
        # so first we enable a shared cache, named SSL (creative!) that is 10mb large
        ssl_session_cache shared:SSL:10m;
        
        # save things in the cache for10 minutes
        # if you're not making a request at least every 10 minutes, this isn't going
        # to accomplish anything anyway
        ssl_session_timeout 10m;
        
        ...
}
```

If you wish to redirect all http traffic to https you can add this additional Nginx server configuration.

```
server{

      listen 80;
      
      server_name [host];

      return 301 https://[host]$request_uri;
}
```

Test config and reload Nginx service.

    sudo nginx -t && sudo service nginx reload
    
# Source

[Nginx converting rewrite rules](http://nginx.org/en/docs/http/converting_rewrite_rules.html)
[Configuring HTTPS servers](http://nginx.org/en/docs/http/configuring_https_servers.html)
[Strong SSL Security on nginx by Raymii](https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html)