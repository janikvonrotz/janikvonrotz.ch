---
title: How to cope, plan and execute
slug: how-to-cope-plan-and-execute
date: 2024-10-25T09:17:38+02:00
categories:
  - Projectmanagement
tags:
  - interview
  - project
  - how-to
images:
  - /images/krazam-microservices.png
draft: false
---
<small>[Title Image Source](https://www.youtube.com/watch?v=y8OnoxKotPQ)</small>

Being self-employed teaches you a lot. Especially how to manage your personal resources.

When dealing with work we develop individual strategies to plan, communicate and execute.

Sharing these strategies is rather difficult. What works well for you might not work for somebody else at all.

So instead of another productivity-hack-post I tried to thing of a better way to share these strategies. This post is an interview with myself.

<!--more-->

Here are the How-do-you-questions:

1. How do you manage your projects, tasks and todos? 
2. How do you plan your tasks and todos?
3. How do you communicate with customers?
4. How do you communicate in the team?
5. How do you cope with the flood of emails?
6. How do you plan your working schedule?
7. How do you manage context switches when coding?
8. How do you link tasks with branches, commits or pull requests?
9. What note-taking system are you using?
10. Are there any AI-tools you recommend?
11. What feature request do you have for the world of working?

And here are the answers:

**1. How do you manage your projects, tasks and todos?**

I have a private and a business Odoo instance. In both cases I use the project app. 

(This is not a sponsored post for Odoo, but nonetheless I will talk a lot about it.)

Making a clear distinction between todos and tasks is very important. I analyse task into multiple todos. Todos are checkboxes that can be ticked off.

![](/images/odoo-task.png)

The task is pinned on a kanban board. The lanes of the kanban board always look like this:

![](/images/odoo-project-kanban.png)

Note the "Postlog" column. It is literally the opposite of backlog. Tasks that are continuously done and never really finished belong there.

**2. How do you plan your tasks and todos?**

In the past I set a deadline on each task. But this became bothersome with an increasing amount of projects and tasks. I found myself always updating deadlines.

Now I simply focus on the ready and in progress columns for a limited amount of projects. The amount of tasks that can be in progress is limited as well. Think of if as a funnel that cannot be increased.

**3. How do you communicate with customers?**

Let's start with what I try to avoid: Instant Messaging. I make a distinction between synchronous and asynchronous communication and also the quality of information you are sending/receiving.

With instant messaging you receive often low quality, high alert and distractful messages. With asynchronous messaging such as mail your receive high quality, not so urgent and silent messages.

Whenever possible I use the Odoo chatter to send mails. It allows to send and receive messages in the context of a task or any other Odoo documents. Here is the task from before:

![](/images/odoo-chatter.png)

**4. How do you communicate in the team?**

The same as before goes here as well. I try to avoid instant messaging. Instead I want to have meaningful conversation in the context of problem. Again the chatter provides me the channels I need. 

For bigger tasks and thoughts I switch to a proposal mode. This is writing a short document such as a specification, problem analysis or RFQ. Then team members are requested to collaborate on this document. 

**5. How do you cope with the flood of emails?**

First of all I use Thunderbird for mailing. It is stable and what I call a boring tech. Boring technologies/products don't change and stay how the inventors intended them to be. 

For transactional mails I have move-rules. Based on the sender the mails ends up in a folder. So I have only mails by humans in my inbox. My inbox usually looks like this:

![](/images/thunderbird-inbox.png)

The "Group by Sort" filter is an awesome feature of Thunderbird. This sorting function shows the urgency of a message.

**6. How do you plan your working schedule?**

I have a private and a business calendar. I use my calendar to plan meetings and work packages.

Every morning and afternoon is planned using 2 or 4 hours slots. Starting the week I fill up the slots in my calendar.

I try to avoid recurring meetings. Having a fixed slot a calendar is a special treatment.

**7. How do you manage context switches when coding?**

For every project I have a `task` script. This gives my the same interface independent of the project type. [Here is an example for a task script](https://github.com/Mint-System/Ansible-Build/blob/main/task).

A lot is happening on GitHub. GitHub sends a lot of notifications, it can be overwhelming. So I only listen to pull-request (PR) review-requests.

**8. How do you link tasks with branches, commits or pull requests?**

Being able to create a git branch from a task is a must have feature for developers. The well-known code forges (GitHub, GitLab, Gitea, Forgejo) provide a project management tool where you can create issues and link them to commits.

However, I currently don't have this option in Odoo. I simply create feature branches in git and create PRs with the GitHub cli.

**9. What note-taking system are you using?**

For this website, my knowledge base, exploring files, basically everything I use Obsidian. On one side Obssidian is very simple. You open a folder as a vault and then you can search, edit and link markdown files. Markdown files are text files, they are supported by many other tools.

On the other side Obsidian can become very complicated and advanced. There are thousands of plugins that allow you to manage everything in Obsidian. Thus you can become very dependent on Obsidian.

Here is an image of the Obsidian graph. It shows how the notes are linked:

![](/images/obsidian-graph.png)

**10. Are there any AI-tools you recommend?**

Well for coding I use [VSCodium](https://vscodium.com/) with [Codeium](https://codeium.com/). Its a very powerful auto-complete, but not much more.

I thinks most AI-tools are good at create boilerplate code and get started with a task. Other than that, they are not so intelligent. Hallucinating stuff or misinterpreting the context can get you sidetracked very easily.

> I use them, but I don't trust them.

**11. What feature request do you have for the world of working?**

When working in a team or other people we often have to ask for status information. What is the status of XYZ? I think software should provide this information. 

It would be nice if there is a protocol and a common format for project management tools/systems to exchange this kind of data.

Almost every project management software is a walled garden. Compared to other software (f.g. git) they do not work well together.

A great inspiration for this feature request is [ForgeFed](https://forgefed.org/). It is a federation protocol for software forges (GitHub, GitLab, Gitea, Forgejo)
