---
title: 'Working with Ansible - Cleanup tasks'
date: 2018-02-26T11:30:13+00:00
author: Janik von Rotz
slug: working-with-ansible-cleanup-tasks
images:
  - /wp-content/uploads/2018/02/Ansible-text-only-e1519640301147.png
categories:
  - Continuous delivery
tags:
  - ansible
  - cleanup
  - environments
  - transition
---
My current project is heavily based on [Ansible](https://www.ansible.com/). We use Ansible to deploy into different environments and domains. The configuration has become quite complex. That is why we need to cleanup a server from time to time. Especially when one has to upgrade services on a server and requires to get rid of the old services first. In this post I would like to show you, how we define and run Ansible cleanup tasks in our project.
<!--more-->

Imagine this simple directory structure:

```
roles/
  basic/
    tasks/
      main.yml
      clean.yml
```

With a very basic task:

**main.yml**

```
- name: install packages
  yum: name={{ item }} state=present
  with_items:
    - nano
    - git
    - vim
```

Now there is a scenario where we need to get rid of these packages before running the main task. For this reason with have the cleanup task:

**clean.yml**

```
- name: uninstall packages
  yum: name={{ item }} state=absent
  with_items:
    - nano
    - git
    - vim
```

It simply does the opposite of the main task.

Now how do we invoke the cleanup task? Well just include the task conditionally like this:

**main.yml**

```
- include: clean.yml
  when: clean

- name: install packages
...
```

Now whenever the main task is executed it will include the cleanup task if the variable *clean* is true. To run a reinstallation use the command: `ansible-playbook _PLAYBOOK_.yml -e clean=true`.

It is also possible to run the cleanup task only by using tags.

```
- include: clean.yml
  when: clean
  tags: clean

- name: install packages
...
```

Remember when you set a tag the tasks **only** using the tag are executed. Run this command to uninstall only: `ansible-playbook _PLAYBOOK_.yml -e clean=true -t clean`.

Of course you should avoid having cleanup task in the first place. Defining a rollback state is not very Ansible like.
