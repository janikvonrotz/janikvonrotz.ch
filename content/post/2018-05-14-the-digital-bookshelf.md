---
id: 4862
title: The digital bookshelf
date: 2018-05-14T09:45:00+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=4862
permalink: /2018/05/14/the-digital-bookshelf/
image: /wp-content/uploads/2018/05/screenshot.png
categories:
  - Blog
tags:
  - book
  - cover
  - epub
  - memories
  - poster
  - tudluk
---
Two years ago I've started to read books on my mobile device. I have skipped the whole e-reader thing and went straight from hard covers to e-books. Many doubt that you can read a book comfortably on the smart phone, but I can tell with certainty that it works well. It  is only a matter of changing your habits.
However, there is one thing I miss. A hard cover in a bookshelf also represents an association with your memories of the book. Whenever I see a book cover I remember the story. This does not work with virtual book covers. Memories of books I've read on my e-book reader fade as they move beyond the display on my phone. To solve this issue I've come up with a simple solution. I want to create a bookshelf poster and pin it to my bedroom wall. With this idea in mind I've started creating a list of books I've read. Now I would like to present you the tool I've built to create beautiful post from this book list.
<!--more-->

The book list is a simple markdown file. Here's an excerpt of my bookshelf file:

**bookshelf.md**

[code]
...

![](https://images.gr-assets.com/books/1433930311l/20873740.jpg)  
Title: Sapiens: A Brief History of Humankind 
Author: Yuval Noah Harari  
ISBN: 0771038518  
Comment: Thought provoking, picturesque, outstanding and simply amazing. If you have not a clue about humanities history and how we built our society, this the book to start with.  
Rating: 10/10  
Finished: 11.04.2018  

...
[/code]

To create a poster from the list I've created **tudluk**. 

Checkout the [GitHub project page of tudluk](https://github.com/janikvonrotz/tudluk).

Tudluk does the following things:

* Read the `bookshelf.md` file
* Download the book covers
* Build a bookshelf html page
* Run a static http server
* Provide the docs to create a screenshot with headless chrome

In result you get a screenshot like this:

[![](https://raw.githubusercontent.com/janikvonrotz/tudluk/master/screenshot.png)](https://raw.githubusercontent.com/janikvonrotz/tudluk/master/screenshot.png)

I will also add a picture of the final poster once it is pinned to the wall.

<a href="https://janikvonrotz.ch/wp-content/uploads/2018/05/the-digital-bookshelf-poster.jpg"><img src="https://janikvonrotz.ch/wp-content/uploads/2018/05/the-digital-bookshelf-poster-1024x576.jpg" alt="" width="720" height="405" class="aligncenter size-large wp-image-4922" /></a>

