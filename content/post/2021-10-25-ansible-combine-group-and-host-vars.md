---
title: "Ansible: Combine group and host vars"
slug: ansible-combine-group-and-host-vars
date: 2021-10-25T08:13:01+02:00
categories:
 - Continuous delivery
tags:
 - ansible
 - tipp
 - tutorial
images:
 - /wp-content/uploads/2018/02/Ansible-text-only-e1519640301147.png
---

In Ansible the configuration parameters are usually separated into two categories: host and group vars. As their names say these are variables that are either applied to a specific host or a group of hosts. And here is already an edge case. What if we want to configure a variable for group, but then also give the possibility to modify for a specific host. Read on to see my solution. 

<!--more-->

This use case occurs usually for list vars. In this example we have a list of packages that need to be installed for a host. We have default group setting the `packages` var, but then there is also a host specific variable.

This is the group var:

**inventory/group_vars/all.yml**

```yml
packages:
  - name: git
  - name: dnsutils
  - name: zsh
```

For the host var prefix the var with  `host_`. This helps to make a distinction and of course not overwriting the group var.

**inventory/host_vars/server1.yml**

```yml
host_packages:
  - name: cifs-utils
```

If the variable is list of dictionaries, it is easy to merge the two variables:

**roles/packages/tasks/main.yml**

```yml
- name: Combine group and host vars
  set_fact:
    packages: "{{ packages + host_packages }}"
  when: host_packages is defined
```

I recommend to define any list as a list of dictionaries.