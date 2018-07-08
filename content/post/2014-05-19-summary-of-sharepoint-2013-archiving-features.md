---
title: Summary of SharePoint 2013 archiving features
date: 2014-05-19T09:25:35+00:00
author: Janik von Rotz
slug: summary-of-sharepoint-2013-archiving-features
dsq_thread_id:
  - "2696211687"
image: /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
---
SharePoint 2013 provides the below types of Archiving

1.	List-based retention and Auditing (List and Libraries)
2.	Records management (Records Archive, In-Place Record)
3.	Site based retention (Applied to Sites and Exchange Server 2013 Team Mail boxes)
<!--more-->
List based Retention Policy Features:

1.	Moving the item to the Recycle Bin
2.	Permanently deleting the item
3.	Transferring the item to another location
4.	Starting a workflow
5.	Skipping to the next stage
6.	Declaring the item to be a record
7.	Deleting all previous drafts of the item
8.	Deleting all previous versions of the item

Site based Retention Policy Features:

1.	New feature introduced in SharePoint 2013
2.	Site Policies used for keeping the growth of sites in control
3.	Sites, Sub-sites and Exchange mailbox associated with a site can be closed or deleted
4.	Closed sites does not appear in places were sites are aggregated – for example, outlook, Outlook Web App, or Project Server 2013 — but users can still modify a closed site and its content by using the URL to reach the site

Site based Retention - Site Policy Options:

1.	Do not close or delete the site automatically (If a policy that has this option is applied to a site, the site owner must delete the site manually)
2.	Delete the site automatically (If a policy that has this option is applied to a site, the site owner may close the site manually, but the site will be deleted automatically)
3.	Close the site automatically and delete the site automatically (This option gives the same choices for how to delete the site automatically, and also requires you to specify how long after its creation
4.	time the site will be closed) Run a workflow to close the site, and delete the site automatically (This option gives the same choices for how to delete the site automatically, and also requires you to specify a workflow to run)

# Source

[Plan for information management policy in SharePoint Server 2013](http://technet.microsoft.com/en-us/library/cc262490(v=office.15).aspx)
[Overview of site policies in SharePoint 2013](http://technet.microsoft.com/en-us/library/jj219569(v=office.15).aspx)