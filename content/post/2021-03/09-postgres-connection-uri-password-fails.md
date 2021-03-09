---
title: "Posgres connection URI password fails"
slug: postgres-connection-uri-password-fails
date: 2021-03-09T14:16:59+01:00
categories:
 - Software development
tags:
 - psql
 - postgres
images:
 - images/PostgreSQL-logo.png
---

With the `psql` command line tool you can either pass the connection credentials as parameters or simply as one connection string.

The general form for a connection URI is:

`postgresql://\[user\[:password\]@\]\[host\]\[:port\]\[,...\]\[/dbname\]\[?param1=value1&...\]`

While developing an Odoo module I could no longer execute queries using connections URIs. I always got the error `psql: error: FATAL:  password authentication failed for user "odoo"`.

<!--more-->

Recently, I switched from MacOS to an Ubuntu-based system. That might be a cause for this behaviour. In this case I simply had to specify the port in the URI explictly.

```bash
➜  Odoo-Development git:(14.0) ✗ psql postgres://odoo:odoo@localhost/odoo
psql: error: FATAL:  password authentication failed for user "odoo"
FATAL:  password authentication failed for user "odoo"

➜  Odoo-Development git:(14.0) ✗ psql postgres://odoo:odoo@localhost:5432/odoo
psql (12.6 (Ubuntu 12.6-0ubuntu0.20.10.1), server 12.5 (Debian 12.5-1.pgdg100+1))
Type "help" for help.

odoo=#
```