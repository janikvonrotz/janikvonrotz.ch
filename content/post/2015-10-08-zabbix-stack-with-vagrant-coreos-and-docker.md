---
title: Zabbix stack with Vagrant, CoreOS and Docker
date: 2015-10-08T16:03:44+00:00
author: Janik von Rotz
slug: zabbix-stack-with-vagrant-coreos-and-docker
dsq_thread_id:
  - "4206142138"
images:
  - /wp-content/uploads/2015/10/Docker-logo.png
categories:
  - Docker
tags:
  - code
  - configuration
  - coreos
  - deploy
  - docker
  - mysql
  - phpmyadmin
  - script
  - ship
  - stack
  - vagrant
  - zabbix
---
This short guide shows you how to set up a virtual Zabbix Network Monitoring Stack with MySQL and CoreOS deployed by Docker.

On your host machine make sure that these tools are available.

* [Vagrant](http://www.vagrantup.com/downloads)
* [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
* [Git](http://git-scm.com/downloads)
<!--more-->

First we are going to setup a virtual machine with Vagrant.

Create a project directory and open navigate there with your command line.
Next clone the CoreOS Vagrant repository.

    git clone https://github.com/coreos/coreos-vagrant/
    cd coreos-vagrant

This repository is preconfigured CoreOS installation.

Open up the `Vagrant` file and edit the lines as showed below.

    $vm_memory = 2024

Now install and run the virtual machine with Vagrant.

    vagrant up

If this command finishes without an error, your virtual machine should be up and running in the VirtualBox environment.

Now connect to the virtual machine with ssh.

    vagrant ssh

Hint: For windows hosts, make sure that the ssh.exe binary that is shipped with Git is available in your system PATH variable.

CoreOS has Docker pre-installed, so no worry about that. We will now deploy our MySQL and Zabbix stack with preconfigured Docker images.

Install and run the MySQL with Docker.

    docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password -d mysql:latest

Install and the phpMyAdmin image.

    docker run -d --name phpmyadmin --link mysql:mysql -p 1234:80 nazarpc/phpmyadmin

Get the configuration of your MySQL container.

    docker inspect mysql

And look for the IPv4 address. We will use in the next statement.

Run the Zabbix image

    docker run \
    -d \
    --name zabbix \
    -p 80:80 \
    -p 10051:10051 \
    -link mysql:db \
    --env="DB_USER=root" \
    --env="DB_PASS=password" \
    --env="SLACK_WEBHOOK=webhook_url" \
    million12/zabbix-server

Now open the browser on your host machine to `http://host_ip` and login with `Admin:zabbix`.

You can get the host machines ip by tipping `ifconfig` on command line.

You phpMyAdmin installation is accessbile under `http://host_ip:1234`.

# Troubleshooting

In case some fails are doesn't work as supposed, follow these steps:

Reboot your virtual machine.

    sudo reboot

Login and start the docker containers.

    docker start container_name

Check if the containers have been started successfully.

    docker ps

# Source

[https://hub.docker.com/r/million12/zabbix-server/](https://hub.docker.com/r/million12/zabbix-server/)