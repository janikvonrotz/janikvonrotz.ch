---
id: 460
title: Exchange Update Offline Address Book
date: 2013-08-27T10:54:43+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=460
permalink: /2013/08/27/exchange-update-offline-adress-book/
dsq_thread_id:
  - "1653822855"
image: /wp-content/uploads/2013/08/exchange-2013-e1393417827333.jpg
categories:
  - Exchange
  - PowerShell
tags:
  - adress
  - book
  - error
  - exchange
  - fail
  - offline
  - update
---
Since Exchange 2010 the graphical console doesn't support the same functionality as the PowerShell Exchange console, it's possible that there occur some exotic errors or a lack of functionality while working with the graphical console. I recommend to use only the Exchange PowerShell console for administrative work.

For example: I had to update the offline address book, I've deleted some distribution groups, updated the address list and the offline book, all with the graphical console. Result the address book still wasn't up to date in the Outlook client after downloading the offline address book.

So I did the same thing with PowerShell:

[code lang="ps"]

Get-OfflineAddressbook | Update-OfflineAddressbook
Get-ClientAccessServer | Update-FileDistributionService

[/code]

And hurray everything worked.