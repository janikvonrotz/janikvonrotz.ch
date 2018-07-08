---
title: Simple Redirect with Apache2
date: 2013-11-15T10:46:47+00:00
author: Janik von Rotz
slug: simple-redirect-with-apache2
dsq_thread_id:
  - "1967892452"
image: /wp-content/uploads/2013/11/apache.jpg
categories:
  - Apache
tags:
  - apache
  - domain
  - helicopter
  - iis
  - ngnix
  - obsolete
  - permantent
  - redirect
  - site
  - webpage
  - webserver
---
It's obvious Apache2 is no more the first choice for a webserver, depending on the requirments <a href="https://www.theorganicagency.com/apache-vs-nginx-performance-comparison/" target="_blank">Ngnix </a>and IIS are now much more attractive.

Maybe you've already updated to Ngnix or IIS want to redirect from the old website to the new website in case Apache is still a dependency.

<!--more-->

The most simple solution to redirect a website domain or url to a new website is this apache virtual host configuration:

```

<VirtualHost *:80>
  ServerName domain.ch
  ServerAlias www.domain.ch
  ServerAdmin webmaster@domain.ch
  ErrorLog /var/log/apache2/domain.ch-error.log
  CustomLog /var/log/apache2/domain.ch-access.log combined
  Redirect permanent / https://www.somewhere.ch/aktuell/whatever/
</VirtualHost>

```

Latest version of this snippet: <a href="https://gist.github.com/7481550" target="_blank">https://gist.github.com/7481550</a>