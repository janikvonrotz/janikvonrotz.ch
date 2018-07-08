---
title: Add Piwik tracking code to your Koken site
date: 2015-08-19T08:38:41+00:00
author: Janik von Rotz
slug: add-piwik-tracking-code-to-your-koken-site
dsq_thread_id:
  - "4046236797"
image: /wp-content/uploads/2015/08/Koken-Logo.png
categories:
  - Koken
tags:
  - cms
  - code
  - enable
  - html
  - injection
  - koken
  - piwik
  - plugin
  - support
  - tracking
  - workaround
---
Sadly there's no native Piwik plugin for the Koken photography CMS.
However you can use another plugin the inject the tracking code to the footer of your site.

* Install the **HTML Injector** plugin (Site > Settings > Plugin > Download Plugins).
* **Enable** the **plugin** (Site > Settings > Plugin > HTML injector > Enable).
* **Setup** the **plugin** (Site > Settings > Plugin > HTML injector > Setup).
* **Copy** the **tracking code** of your Piwik site (Piwik > Administration > Websites > [site] > View Tracking code).
* **Paste** the **code** into the **footer field** and save it.

Well done, your visitors are tracked now.

# Source
[lars-mielke.de - Piwik Integration in Koken](http://lars-mielke.de/4333/piwik-integration-in-koken/)