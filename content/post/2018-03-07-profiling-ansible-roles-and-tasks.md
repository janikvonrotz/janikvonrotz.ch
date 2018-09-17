---
title: Profiling Ansible roles and tasks
date: 2018-03-07T13:11:25+00:00
author: Janik von Rotz
slug: profiling-ansible-roles-and-tasks
specific_page_layout:
  - default-sidebar
images:
  - /wp-content/uploads/2018/02/Ansible-text-only-e1519640301147.png
categories:
  - Ansible
tags:
  - ansible
  - benachmark
  - callback
  - measure
  - performance
  - plugin
  - profile
  - roles
  - tasks
---
Performance is critical when deploying an environment with Ansible. By default Ansible does not tell how much time elapsed for specific role or task. However, this information would be critical to identify inefficient tasks. Luckily Ansible offers an interface for callback plugins. With the help of a callback plugin one can hook into the role or task execution call. In this post I'll show you how to configure the callback plugins for profiling roles and tasks. It is quite easy.
<!--more-->

First create a plugin folder in your Ansible project.

`mkdir callback_plugins`

Step into the directory and download the plugins from the Ansible repo.

```
cd callback_plugins
curl https://raw.githubusercontent.com/ansible/ansible/devel/lib/ansible/plugins/callback/profile_tasks.py -o profile_tasks.py
curl https://raw.githubusercontent.com/ansible/ansible/devel/lib/ansible/plugins/callback/profile_roles.py -o profile_roles.py
```

To enable the plugins update the Ansible configuration file.

**ansible.cfg**

```
[defaults]
callback_whitelist = profile_tasks, profile_roles
```

Now whenever you run a playbook the finish report will tell how much time elapsed for each task and role.

Here is an example of the profile roles results:

```
=============================================================================== 
adnwildfly ------------------------------------------------------------- 54.19s
couchbase-setup -------------------------------------------------------- 52.25s
nevisproxy ------------------------------------------------------------- 38.35s
testdata-loader -------------------------------------------------------- 36.97s
keybox ----------------------------------------------------------------- 22.56s
keybox-slot ------------------------------------------------------------ 16.25s
nevisauth -------------------------------------------------------------- 15.55s
nevislogrend ----------------------------------------------------------- 15.29s
setup ------------------------------------------------------------------- 8.66s
users ------------------------------------------------------------------- 8.03s
nevissaml --------------------------------------------------------------- 3.72s
spool ------------------------------------------------------------------- 3.32s
reference-rp ------------------------------------------------------------ 1.92s
test-tools -------------------------------------------------------------- 1.01s
release ----------------------------------------------------------------- 0.62s
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
total ----------------------------------------------------------------- 278.69s
```