---
title: "Nginx WAF with ModSecurity and OWASP CRS"
slug: nginx-waf-with-modsecurity-and-owas-crs
date: 2020-02-26T08:57:31+01:00
categories:
 - Security
tags:
 - nginx
 - proxy
 - web application firewall
 - modsecurity
 - owasp
 - core set rules
images:
 - /images/owasp modsecurity core rule set.png
---

This tutorial explains how to enable and test the Open Web Application Security Project Core Rule Set (OWASP CRS) for use with the Nginx and ModSecurity. We are going to setup a Docker Compose project and deploy a ModSecurity enabled Nginx container with the CRS. Everything will be done using Open Source tools only.
<!--more-->

# Terms

For better understanding of what is going on here we have to define some terms.

**Nginx**

Nginx is the most popular web server. It can serve static content, process https requests and do much more.

[Nginx - Homepage](https://nginx.org/)

**WAF**

Means web application firewall. Compared to normal firewalls WAFs do not protect internet traffic (ISO layer 3 and 4) but protect http/s traffic (layer 7).

**ModSecurity**

ModSecurity is an open source, cross-platform web application firewall module.

[ModSecuirty - Homepage](https://modsecurity.org/)

**OWASP**

OWASP is a non-profit organization that works to improve the security of software.

[OWASP - Homepage](https://owasp.org/)

**CRS**

Core Rule Set (CRS) is a set of generic attack detection rules for use with ModSecurity or compatible web application firewalls.

[OWASP - ModSecurity Core Rule Set](https://coreruleset.org/)

# Prerequisites

This guide assumes that [Docker](https://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/) is installed and you know your way around Git, Docker containers, Bash, web servers and log files.

# Files

Our example projects consists of various files. Here is a tree view of what we are going to create:

```bash
nginx-modsecurity-crs
├── docker-compose.yml
└── etc
    ├── modsecurity
    │   ├── RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf
    │   ├── crs # git submodule
    │   └── crs-setup.conf
    ├── modsecurity.d
    │   ├── include.conf
    │   └── modsecurity.conf
    └── nginx
        └── conf.d
            └── default.template
```

Step by setp we are going to add these files and finally deploy and test the Nginx WAF.

If you wanna shortcut the walkthrough head over to the [GitHub repository](https://github.com/janikvonrotz/nginx-modsecurity-crs).

First we are going to setup our project repository, include the CRS and pin its version.

```bash
cd ~
mkdir nginx-modsecurity-crs && cd nginx-modsecurity-crs
git init
git submodule add https://github.com/SpiderLabs/owasp-modsecurity-crs.git etc/modsecurity/crs
cd etc/modsecurity/crs/
git checkout v3.2.0
cd ~/nginx-modsecurity-cr
```

Let's get startet with the Nginx config file.

**etc/nginx/conf.d/default.template**

```conf
server {
    listen 443 ssl;
    server_name  ${SERVERNAME};
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS;
    ssl_certificate /etc/nginx/conf/server.crt;
    ssl_certificate_key /etc/nginx/conf/server.key;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

This file configures a simple tls secured webserver for testing.

**etc/modsecurity.d/modsecurity.conf**

Copy the `modsecurity` file from the [code repository](https://github.com/janikvonrotz/nginx-modsecurity-crs/blob/master/etc/modsecurity.d/modsecurity.conf).

**etc/modsecurity.d/include.conf**

```conf
# Include the recommended configuration
Include /etc/modsecurity.d/modsecurity.conf
# OWASP CRS v3 rules
Include /etc/modsecurity/crs-setup.conf
Include /etc/modsecurity/crs/rules/*.conf
Include /etc/modsecurity/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf
```

By default we include all OWASP CRS rules.

**etc/modsecurity/crs-setup.conf**

Copy the `crs-setup.conf` file from the [code repository](https://github.com/janikvonrotz/nginx-modsecurity-crs/blob/master/etc/modsecurity/crs-setup.conf).

**etc/modsecurity/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf**

```conf
# https://github.com/SpiderLabs/owasp-modsecurity-crs/blob/v3.2/dev/rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf.example
#
# Examples:
# SecRuleRemoveById 942100
# SecRuleRemoveByTag "attack-sqli"
# SecRuleUpdateTargetById 942100 "!ARGS:password"
# SecRuleUpdateTargetByTag "attack-sqli" "!ARGS:password"
```

This file is required if you want to disable false positives. See the next chapter for more details.

**docker-compose.yml**

```yml
version: "3"
services:
  waf:
    image: owasp/modsecurity:3-nginx
    ports:
      - "443:443"
    volumes:
      - "$PWD/etc/modsecurity:/etc/modsecurity"
      - "$PWD/etc/modsecurity.d/include.conf:/etc/modsecurity.d/include.conf"
      - "$PWD/etc/modsecurity.d/modsecurity.conf:/etc/modsecurity.d/modsecurity.conf"
      - "$PWD/etc/nginx/conf.d/default.template:/etc/nginx/conf.d/default.template"
    environment:
      - SERVERNAME=localhost
    command: /bin/bash -c "envsubst '$$SERVERNAME' < /etc/nginx/conf.d/default.template > /etc/nginx/conf.d/default.conf && exec nginx -g 'daemon off;'"
```

This is the Docker Compose file that spins up a Nginx container with ModSecurity and the CRS.

This is all we need. Run Docker Compose and head over to the Test chapter.

```bash
docker-compose up -d
```

# Test

By default the Nginx container starts in audit mode. Before enabling the security engine you want to ensure that ModSecurity does not block any false positives. Therefore you evalute your webapp in audit mode.

Let's walk through an audit.

Tail the audit log.

```bash
docker exec -it nginx-modsecurity-crs_waf_1 tail -f /var/log/modsec_audit.log
```

Trigger a security rule with curl.

```bash
curl -I 'https://localhost/?param="><script>alert(1);</script>' --insecure
```

The request has not been blocked and you should get a response like this:


```html
HTTP/1.1 200 OK
Server: nginx/1.15.12
Date: Wed, 26 Feb 2020 13:42:17 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 16 Apr 2019 13:08:19 GMT
Connection: keep-alive
ETag: "5cb5d3c3-264"
Accept-Ranges: bytes
```

The output of the audit log looks like this:

```txt
ModSecurity: Warning. detected XSS using libinjection. 
[file "/etc/modsecurity/crs/rules/REQUEST-941-APPLICATION-ATTACK-XSS.conf"]
[line "37"] [id "941100"] [rev ""] [msg "XSS Attack Detected via libinjection"] 
[data "Matched Data: XSS data found within ARGS:param: "><script>alert(1);</script>"] 
[severity "2"] [ver "OWASP_CRS/3.2.0"] [maturity "0"] [accuracy "0"] [tag "application-multi"] 
[tag "language-multi"] [tag "platform-multi"] [tag "attack-xss"] [tag "OWASP_CRS"] 
[tag "OWASP_CRS/WEB_ATTACK/XSS"] [tag "WASCTC/WASC-8"] [tag "WASCTC/WASC-22"] 
[tag "OWASP_TOP_10/A3"] [tag "OWASP_AppSensor/IE1"] [tag "CAPEC-242"] [hostname "172.22.0.1"] 
[uri "/"] [unique_id "158272291834.052399"] 
[ref "v12,28t:utf8toUnicode,t:urlDecodeUni,t:htmlEntityDecode,t:jsDecode,t:cssDecode,t:removeNulls"]
```

An XSS attack has been detected by rule number `941100`.  
Now you would decide wether to disable this rule by updating the `etc/modsecurity/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf` file or update your webapp.

If your webapp has been tested and the audit log does not have any new entries, the security engine can be enabled.

Edit the ModSecuity config to do so.

**etc/modsecurity.d/modsecurity.conf**

```
...
SecRuleEngine On
...
```

Restart the Nginx container.

```
docker-compose restart
```

Trigger the security rule.

```bash
curl -I 'https://localhost/?param="><script>alert(1);</script>' --insecure
```

And you should get a response like this:

```html
HTTP/1.1 403 Forbidden
Server: nginx/1.15.12
Date: Wed, 26 Feb 2020 13:35:17 GMT
Content-Type: text/html
Content-Length: 154
Connection: keep-alive
```

The request has been blocked succesfully.

Now you can easily setup a first grade WAF without depending on any third party product. It's that easy.

# Next

If want to learn more aobut the OWASP ModSecurity CRS head over to the [project site](https://owasp.org/www-project-modsecurity-core-rule-set/). There you'll find detailed documentations and tutorials.
