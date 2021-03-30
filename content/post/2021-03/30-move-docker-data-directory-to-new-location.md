---
title: "Move Docker data directory to new location"
slug: move-docker-data-directory-to-new-location
date: 2021-03-30T22:30:50+02:00
categories:
 - Unix
tags:
 - docker
 - deamon
images:
 - /wp-content/uploads/2015/10/Docker-logo.png
---

The standard data directory used for docker is `/var/lib/docker`, and since this directory will store all your images, volumes, etc. it can become quite large.
<!--more-->

Follow the steps below to move the Docker data directory to a new location. This makes especially sense if you want to avoid running out of disk space on your root partition.

Stop the Docker deamon.

```bash
sudo service docker stop
```

Create a config file **/etc/docker/daemon.json**

```json
{ 
   "data-root": "/path/to/your/docker" 
}
```

Copy the current directory to the new directory.

```bash
sudo cp -rp /var/lib/docker /path/to/your/docker
```

Rename the old directory.

```bash
sudo mv /var/lib/docker /var/lib/docker.old
```

Restart the Docker deamon.

```
sudo service docker start
```

Test if all services work as expected.

If everything is good, remove the old Docker directory.

```
sudo rm -rf /var/lib/docker.old
```