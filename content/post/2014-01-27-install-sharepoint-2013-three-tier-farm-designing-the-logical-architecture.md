---
title: 'Install SharePoint 2013 Three-tier Farm - Designing the Logical Architecture'
date: 2014-01-27T09:10:29+00:00
author: Janik von Rotz
slug: install-sharepoint-2013-three-tier-farm-designing-the-logical-architecture
images:
  - /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - architecture
  - sharepoint
  - site templates
  - three tier
---
*This post of is part of my [Install SharePoint 2013 Three-tier Farm](https://janikvonrotz.ch/projects/install-sharepoint-2013-three-tier-farm/) project.*

I assume that the reader of this tutorial has already defined the business requirements for this SharePoint installation.

The physical architecture depends on the requirements of the logical design. That's why you have to define you logical architecture first.

<!--more-->

The logical architectures unites the business requirements and the possibilities of SharePoint to set up multiple site collections with different urls, templates and databases.

The easiest way to write down these resources is a table or a note like this:

<strong>Intranet</strong>

Site collection: sharepont.domain.com
Hostname: https://sharepoint.domain.com
Site url: /
Template: Blank Sites
Database: SP_Content_Intranet

<strong>Search</strong>

Site collection: sharepont.domain.com
Hostname: https://sharepoint.domain.com
Site url: /search
Template: Enterprise Search Center

<strong>My Site</strong>

Site collection: mysite.domain.com
Hostname: https://mysite.domain.com
Site url: /
Template: User Profile
Database: SP_Content_MySite

<strong>IT Wiki</strong>

Site collection: sharepont.domain.com
Hostname: https://sharepoint.domain.com
Site url: /itwiki
Template: Enterprise Wiki
Database: SP_Content_ITWiki

<strong>Extranet VR</strong>

Site collection: extranetvr.domain.com
Hostname: https://extranetvr.domain.com
Site url: /
Template: Blank Sites
Database: SP_Content_ExtranetVR

<strong>Extranet VR - Search</strong>

Site collection: extranetvr.domain.com
Hostname: https://extranetvr.domain.com
Site url: /search
Template: Enterprise Search Center