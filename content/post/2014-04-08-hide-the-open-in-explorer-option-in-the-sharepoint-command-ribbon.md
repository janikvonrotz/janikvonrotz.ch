---
title: Hide the Open in Explorer option in the SharePoint command ribbon
date: 2014-04-08T08:27:02+00:00
author: Janik von Rotz
slug: hide-the-open-in-explorer-option-in-the-sharepoint-command-ribbon
images:
  - /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - explorer
  - sharepoint
  - solution
---
Since it's possible to upload documents to SharePoint via Drag and Drop, the "Open in Explorer" option in SharePoint can be removed.

I recommend to remove this option unregarded of it's possible to upload via Drag and Drop as the user can't update document metadata in the windows explorer.

Here's an example of what we want:

[![Hidden Open in Explorer option](/wp-content/uploads/2014/04/Hidden-Open-in-Explorer-option.jpg)](https://janikvonrotz.ch/2014/04/08/hide-the-open-in-explorer-option-in-the-sharepoint-command-ribbon/hidden-open-in-explorer-option/)

And the solution to achieve this, is available on GitHub: <a href="https://codeberg.org/janikvonrotz/SharePoint-Hide-Open-With-Explorer">https://codeberg.org/janikvonrotz/SharePoint-Hide-Open-With-Explorer</a>