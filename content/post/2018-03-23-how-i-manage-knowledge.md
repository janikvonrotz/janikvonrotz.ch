---
title: How I manage knowledge
date: 2018-03-23T16:40:48+00:00
author: Janik von Rotz
slug: how-i-manage-knowledge
specific_page_layout:
  - default-sidebar
images:
  - /wp-content/uploads/2018/03/Knowledge-Book-Site.jpg
categories:
  - Blog
tags:
  - knowledge base
  - knowledge management
  - problem
  - solution
  - strategy
  - taxonomy
---
So far I mostly posted about technical aspects of IT. Mainly solutions for particular problems. In future I would like to change that a bit. Not only because I am currently working on confidential projects, but also because I believe I have developed some ideas worth to be shared. 
One of those ideas is about knowledge management. While I never dealt with the topic on a research level, I made some experience while applying knowledge base solutions in different companies. Interest in this topic is probably cause because I often faced a frustrating situation.
IT employees at company X were not incentivized to write documentations at all. Company X had not institutionalized a proper knowledge management. In result with employees leaving the company, knowledge got lost. Mistakes were repeated. Frustration increased. A vicious cycle.
<!--more-->

## It is not that easy

To solve this problem I have introduced and applied different tools to harvest and spread knowledge. One of those tools was SharePoint. SharePoint is a collaboration platform by Microsoft. It introduces advanced tools for every aspects of collaboration. However, using SharePoint comes at the cost of a [vendor lock in](https://ben.balter.com/2012/05/10/free-yourself-from-the-tyranny-of-sharepoint/). In that way I learned that there are extensive solutions to manage knowledge in the business environment, but at the same time many of the existing solutions do not support the idea of having knowledge data vendor-independent.

## Personal approach

To be more capable at work and having more knowledge at hand, I have tried a variety of note taking tools (Evernote, OneNote, Google Notes, ...).
Most of them were insufficient. They either stored my content in a proprietary data format or did not provide the functionality required for my daily work.
So I developed a personal workflow for managing knowledge. I will discuss my solution later on. First, I want to present my 4 rules of knowledge management.

## Knowledge requires taxonomy

Taxonomy is the definition of a classification or categorization. Having a tree based structure to store knowledge makes it easier to recall specific content. Taxonomy helps to filter, navigate and explore the content of a knowledge base. Moreover, a company wide standardized taxonomy defines terms and thus makes it easier to communicate. Knowledge content must be stored in explicitly defined tree based data structure.

## Knowledge needs to be accessible

Thinking happens immediately. Usually you can recall something within seconds or never. That is why access to a knowledge base content must happen fast. It all comes down to search performance. A knowledge base system must act reactive and provide the most relevant search results.

## Knowledge documents must be platform-independent

Knowledge has become the most important resource for companies. Money, material and employees have become a commodity. It is important that knowledge is available after years, moves on with the company and stays relevant. This is also true with personal knowledge. You do not want store content in software that might do not exist in the near future.

## The foundation of Knowledge management is encouragement

I personally believe that economic behavior incentives employees to not contribute to a knowledge base at all. People judge the effort of contribution subjectively and treat it as unnecessary work if they can not benefit career wise. That is why employees have to be incentivized to contribute in different ways. A very powerful strategy is the implementation of gamification. Rewarding people with virtual gifts and reputation has been proofed to be suitable solution.

## This is how I store knowledge

For assistance at my daily work, I have developed a distinct workflow and toolbox to manage my personal knowledge. To be more precise the following technologies are involved:

* Visual Studio Code
* Git
* Markdown

Using these tools I try to document as much as possible. Every document version is tracked via git. For every domain of my life (education, private, business) there is a knowledge base. The knowledge bases are stored in a git repository and thus can be made shared with other parties. Here is an example of working with this toolbox:

![Untitled](/wp-content/uploads/2018/03/Knowledge-Base-Visual-Studio-Code.gif)

To force a taxonomy I have defined a guideline for the folder structure: [https://gist.github.com/janikvonrotz/a9eeb4d5ae551073fda8b5d3743b2cfc](https://gist.github.com/janikvonrotz/a9eeb4d5ae551073fda8b5d3743b2cfc)

