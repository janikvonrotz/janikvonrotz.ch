---
title: 'Chatbot dialog design - a best practice proposal'
date: 2016-09-16T17:38:22+00:00
author: Janik Vonrotz
slug: chatbot-dialog-design-a-best-practice-proposal
images:
  - /wp-content/uploads/2016/09/UX-banner-e1474038570257.jpg
categories:
  - Cognitive computing
---
While working on a new chatbot I had to come up with good examples to start and led a conversation with the chatbot. For this I analyzed other bots, tracked the steps it takes to achieve a specific goal and kept notes on repeating patterns. In result me and my team came up with a few basic principles to design dialogs in conversational bots.
<!--more-->
Assuming you already got in touch with common chatbot platform here are the principles:

* A chatbot should understand and appropriately react to keywords at any point in the dialog, such as “hi”, “cancel”, “menu” or “start over”.
* A chatbot reaction should contain a description and alternative options. These option should either provide a help to finish the current dialog or move within the whole conversation forward and backward. 
* Hints on inputs should be offered.
* Carousels should only display results, not menu options.
* New users should get a general introduction on what the bot is about (once).
* The number of steps to complete a task should be kept as small as possible.
* The possibility to abort a dialog should be offered at any point of the dialog.
* Rich text elements, such as cards, should be used carefully. They use a lot of space and might move beyond the viewport.
* Experienced users should have the option to reach their goal very quickly.
* Give suggestions or inspirations for undecided users.
