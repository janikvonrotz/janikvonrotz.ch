---
title: "Pin and update Ubuntu packages with Ansible"
slug: pin-and-update-ubuntu-packages-with-ansible
date: 2020-06-05T20:07:00+02:00
categories:
 - Software configuration management
tags:
 - Ansible
 - patching
 - Ubuntu
 - RedHat
images:
 - "/images/server rack.jpg"
---

Patching is an essential part in server maintenance. For Windows servers and clients you have [WSUS](https://de.wikipedia.org/wiki/Windows_Server_Update_Services) to coordinate updates and security patches. But how do you patch Ubuntu, Debian, RHEL or Fedora machines? The linux distro field is very dispersed, how do you do patch all these systems? In this post I will show you how I install package updates and ensure that specific packages cannot be updated using version pinning.
<!--more-->

The current state of patch managenent.

#  Enterprise derivatives

RHEL and Ubuntu are both enterprise derivates, which means that enterprise features for these distros are managed by companies and not communities.

Ubuntu is managed by [Canonical](https://canonical.com/) and they have their [Livepatch Service](https://ubuntu.com/livepatch) to apply patches.

RHEL is managed by [Red Hat](https://www.redhat.com/) and they provide [Satellite](https://www.redhat.com/de/technologies/management/satellite) to apply patches, system configurations and much more.

# Community derivates

Debian, Fedora and [many more](https://en.wikipedia.org/wiki/List_of_Linux_distributions) are distros managed by communities. If you search their name in combination with patching you end up with various blog posts, where people show how they install the kernel updates and security patches. It is obvious that patch management is executed by companies running a farm of servers and there for seek automated solutions.

However, with Ansible you can easily manage you server farm and therefore also require an advanced solution to apply patches for your hosts.

# Patch Ubuntu using Ansible

First let me assure that the method for patching can also be used for other distros. For better understanding I will show how its done using Ubuntu.

In this tutorial I will show you how I install essential packages and apply system patches. For my system configurations I try to be as explicit as possible, which means I pin versions for installed software. Assume that we have a Docker installation on an Ubuntu server and want to make sure that Docker is not updated as we do not want break compatibility.

## Project setup

In my Ansible project I have created the following inventory and role:

```txt
inventories/setup
├── group_vars
│   ├── all.yml
│   └── ubuntu1804.yml
├── host_vars
│   ├── server1.yml
│   └── server2.yml
└── hosts.yml

roles/update/tasks
├── main.yml
└── update-ubuntu.yml

roles/docker/tasks
├── main.yml
└── install.yml

setup.yml
```

The `setup.yml` is an Ansible playbook that installs docker and updates servers. Note that I simplified the project setup for this tutorial.

## Group vars

The group vars usually contains a list of packages.

**inventories/setup/group_vars/ubuntu1804.yml**

```yml
docker_package: "docker-ce=5:19.03.8~3-0~ubuntu-bionic"
```

I will demonstrate the pin version feature using this package.

## Host vars

The host vars simply tell wether this host should be updated or not.

**inventories/setup/host_vars/server1.yml**

```yml
update: yes
```

## Setup role

The setup role installs docker ce and pins its version if the package string contains an `=`.

**roles/docker/tasks/install.yml**

```yml
- name: Unpin docker-ce version
  dpkg_selections:
    name: "{{ docker_package }}"
    selection: hold
  when: "'=' not in docker_package"

- name: Update apt and install docker-ce
  apt:
    update_cache: yes
    name: "{{ docker_package }}"

- name: Pin docker-ce version
  dpkg_selections:
    name: "{{ docker_package.split('=')[0] }}"
    selection: hold
  when: "'=' in docker_package"
```

Once a package has been pinned, it cannot be updated by the package manger.

## Update role

The update role runs only if updates are enabled for the host.

**update/tasks/main.yml**

```yml
- name: Include update tasks
  include_tasks: update-ubuntu.yml
  when: update
```

The update role refreshes the repo cache and runs a dist upgrade.

**update/tasks/update-ubuntu.yml**

```yml
- name: Update apt-get repo and cache
  apt:
    update_cache: yes
    force_apt_get: yes
    cache_valid_time: 3600

- name: Upgrade all packages
  apt:
    upgrade: dist

- name: Check if reboot is required
  stat: 
    path: /var/run/reboot-required
  register: reboot_required

- name: Reboot system if required
  reboot:
    msg: Rebooting to complete system upgrade
    reboot_timeout: 120
  when: reboot_required.stat.exists
```

If a reboot is required it will do so.

## Run the playbook

The Ansible playbook `setup.yml` supports two running modes. The default mode and the update mode (`update` tag is required).

```yml
- hosts: all
  become: true
  roles:
  - role: docker
    tags: docker
  - role: update
    tags: update,never
```

To install docker on the target host you would enter: `ansible-playbook -i inventory/setup setup.yml -l server1`

And to update the target host you would enter: `ansible-playbook -i inventory/setup setup.yml -l server1 -t update`

It is recommend the run the update command periodically.

I hope I was able to demonstrate how you can manage patches for your linux systems using Ansible. As mentioned this method can be used for other non-aptitude-enabled distros.
