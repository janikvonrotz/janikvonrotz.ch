---
title: "Open Source Software = More Security"
slug: open-source-software-more-security
date: 2022-03-07T10:53:32+01:00
categories:
 - open source
tags:
 - oss
 - security
images:
 - /images/open-source-software.png
---

This post has been translated from  [Mint System - Open Source Software = Sicherheit](https://www.mint-system.ch/blog/mint-system-blog-1/open-source-software-sicherheit-33).

> Why is open source software more secure than closed source software?

Again and again we are confronted with the argument that open source software (OSS) cannot be secure because it is free. The ["there is no free lunch"](https://en.wikipedia.org/wiki/There_ain%27t_no_such_thing_as_a_free_lunch) idiom is also often used. We will explain here why this argument is wrong.

<!--more-->

First of all, it is important to know that software code is not comparable to tangible products. Software code has a special feature - it can be duplicated without considerable effort. This is rather difficult with a car, for example. Each production of a car costs the same. Code, on the other hand, is written once. So if you want to sell software, you can only do so through licensing. You sell the intellectual property, so to speak, but not the program lines.

> Software code has a special feature - it can be duplicated without effort.

This raises the question: What is the value of software code if it can be easily duplicated? Our answer is simple. The value of software code is reflected in the number of systems / users that use and execute the code.

> The value of software code is in the number of executions.

Therefore, the value of code increases with the number of systems / users that execute it. And that is exactly why most code is on collaboration platforms like [GitHub](https://github.com/) or [GitLab](https://about.gitlab.com/). You can find the code for millions of software systems there, for example the [code for the Linux operating system](https://github.com/torvalds/linux), which [made it to Mars](https://www.theverge.com/2021/2/19/22291324/linux-perseverance-mars-curiosity-ingenuity).

These platforms are used by security researchers to find security holes in the code. An ecosystem has emerged where developers are paid to find and close security holes (see for example [hackerone](https://www.hackerone.com)). Disclosing the code therefore creates more security.

> Transparency creates security.

The opposite is called "security through obscurity". This approach is [no longer recommended by security researchers](https://en.wikipedia.org/wiki/Security_through_obscurity#Criticism).

In summary, the following conclusions can be drawn:
* The value of software code lies in the number of executions.
* The number of executions increases when the code is on open platforms.
* There are incentives to search the code on these platforms for security vulnerabilities and to close them.
* Open source code is more secure than closed source code.
