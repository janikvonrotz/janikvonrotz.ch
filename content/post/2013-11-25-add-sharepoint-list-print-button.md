---
id: 769
title: Add SharePoint List Print Button
date: 2013-11-25T09:32:03+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=769
permalink: /2013/11/25/add-sharepoint-list-print-button/
dsq_thread_id:
  - "1997246600"
image: /wp-content/uploads/2013/07/SharePoint-2013-Logo.png
categories:
  - SharePoint
tags:
  - add
  - button
  - content
  - editor
  - print
  - sharepoint
  - webpart
---
You can print every SharePoint list item with the keyboard short cut Ctrl+P. But adding a real print button to the SharePoint list view isn't that difficult too.

Choose the form type in you list options under form webparts.

[![SharePoint List Print Button 1](https://janikvonrotz.ch/wp-content/uploads/2013/11/SharePoint-List-Print-Button-1.png)](https://janikvonrotz.ch/wp-content/uploads/2013/11/SharePoint-List-Print-Button-1.png)

<!--more-->

Add a content editor webpart on top of the list view or right under the view.

[![SharePoint List Print Button 2](https://janikvonrotz.ch/wp-content/uploads/2013/11/SharePoint-List-Print-Button-2.png)](https://janikvonrotz.ch/wp-content/uploads/2013/11/SharePoint-List-Print-Button-2.png)

Click to add new content.

[![SharePoint List Print Button 3](https://janikvonrotz.ch/wp-content/uploads/2013/11/SharePoint-List-Print-Button-3.png)](https://janikvonrotz.ch/wp-content/uploads/2013/11/SharePoint-List-Print-Button-3.png)

Now edit the content as html source.

[![SharePoint List Print Button 4](https://janikvonrotz.ch/wp-content/uploads/2013/11/SharePoint-List-Print-Button-4.png)](https://janikvonrotz.ch/wp-content/uploads/2013/11/SharePoint-List-Print-Button-4.png)

Add this snippet.

```js
`
<input type="button" value=" Print " onclick="window.print();return false;" />
`
```

[![SharePoint List Print Button 5](https://janikvonrotz.ch/wp-content/uploads/2013/11/SharePoint-List-Print-Button-5.png)](https://janikvonrotz.ch/wp-content/uploads/2013/11/SharePoint-List-Print-Button-5.png)

And you get a nice print button.