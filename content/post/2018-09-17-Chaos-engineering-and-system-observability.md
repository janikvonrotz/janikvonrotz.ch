---
title: "System Observability and Chaos Engineering"
slug: 2018-09-17-system-observability-and-chaos-engineering
date: 2018-09-16T14:58:02+02:00
categories:
 - Work
tags:
 - chaos
 - engineering
 - observability
 - resilience
 - distributed systems
 - monitoring
 - unpredictability
images:
  - /images/rope meshwork.jpg
---

At the last [Jazoon tech days](https://jazoon.com/) I learned about the current state of DevOps and other related topics. During the talks "chaos engineering" and "observability" were mentioned many times. I didn't know either and became curious what it was about.
<!--more-->

# Observability

TL:DR; Monitoring tells you whether a system is working, observability lets you ask why it isn't working.

A very insightful presentation on observability was given by [Charity Majors](https://twitter.com/mipsytipsy).

{{< youtube oGC8C9z7TN4 >}}

In her talk she covered the shortcomings of traditional metrics and logs. She pointed out that monitoring as we know it is dead (including dashboards), and that we need to focus on observability.

Enhancing system observability for a software project requires a culture shift rather than implementing a new technology. Software projects become more complex and thus we cannot detect and prevent any kind of error that occurs in the future. Increased observability helps finding, debugging and fixing an issue more easily.

On an abstract level the following question provides a good viewpoint on what observability is about.

> Can you understand what's happening inside your system, just by asking questions from the outside?  
> -- Charity Majors

If the system is a blackbox it becomes very difficult to understand its behavior. That might sound obvious, but of course it is impossible to build a complex system without unknown components.

One of the drivers that makes monitoring obsolete and supports the idea of observability is the unpredictability of system failures. The next quotes puts it very well.

> Distributed systems haven an infinitely long list of almost-impossible failure scenarios that make staging environments particularly worthless.  
> -- Charity Majors

This fact has probably lead us to the discipline of Chaos Engineering.

# Chaos Engineering

TL:DR; Chaos Engineering is the discipline of experimenting on a distributed system in order to build confidence in the system’s capability to withstand turbulent conditions in production.

Aaron Blohowiak worked as a Chaos Engineer at Netflix and showed how their multi-region infrastructure deals with outages.

{{< youtube nkndbc_Qp7Q >}}

The fundamental idea of chaos engineering is already part of the company's culture.

> In general, freedom and rapid recovery is better than trying to prevent error. We are in a creative business, not a safety-critical business.  
> -- jobs.netflix.com/culture

During a back stage talks Aaron talked about Chaos Engineering in practice. The goal of Chaos Engineering is the improvement of system resilience. Instead of practicing error prevention and you focus on running experiments with your system and make sure that it is able to recover from failures.

> Due to the inevitability of errors, investing in isolation, recovery and remediation is superior to investing in prevention.  
> -- Aaron Blohowiak

Once system resilience improves, the system can withstand unpredictable errors in production.

If you are interested in this topic start reading the [Principles of Chaos Engineering](https://principlesofchaos.org/) manifesto.

Hope you learned as much as I did! 😃💡 Otherwise leave a question in the comments!