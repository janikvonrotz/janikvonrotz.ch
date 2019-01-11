---
title: "Open multiple projects in Intellij"
slug: open-multiple-projects-in-intellij
date: 2019-01-11T10:01:30+01:00
categories:
 - Software development 
tags:
 - intellij
 - java
 - gradle
images:
 - /images/intellij idea logo.png
---

Most IDEs provide workspaces that contain multiple projects and thus enable you to work on mutliple projects in one instance of the IDE.
IntelliJ which has became the defacto standard for Java Devs does not support workspaces.
So how is it possible to open mutliple projects in one IntelliJ instance?
<!--more-->

Well, it is quite simple. In context of IntelliJ you can replace the idea of wokspaces with projects and projects with modules. An IntelliJ project can contain multiple modules.

Let's put this to practice. I assume you have checked out multiple project, lets say at `/local/projects/`.  
Create a new project called *root.* at `/local/projects/root` in IntelliJ.
Next add a new module from an existing source:

* File > New > Module from Existing Sources...
* Select the project e.g. `/local/projects/projectX`
  * If available select the buid.gradle or another file that indicates the project model.
* Finish the wizard

Now the module/project should be loaded on the top level of the root project. Depending the projec model IntelliJ should detect features and configure it accordingly.

Any questions?

Source: [StackExchange - Superuser](https://superuser.com/questions/850373/does-intellij-idea-have-the-concept-of-workspace-similar-to-eclipse-with-multi)