---
title: "Backup Docker volumes with this script"
slug: backup-docker-volumes-with-this-script
date: 2020-08-18T21:38:34+02:00
categories:
 - Scripting
tags:
 - backup
 - docker
 - script
images:
 - /images/shipping container.jpg
---

The official Docker documentation has [section about container backup](https://docs.docker.com/storage/volumes/#backup-restore-or-migrate-data-volumes). This section tells you how to backup an attached volume and the command to do so is pretty elaborate. For automating backups you don't want to script parametrized commands such as this one. A simple script will do the job.
<!--more-->

Here is a script that I use to backup all mounted volumes from a running container.

**docker-volume-backup**

```bash
#!/bin/bash

# Exit script if command fails
set -e

# Display Help
Help() {
  echo
  echo "docker-volume-backup"
  echo "####################"
  echo
  echo "Description: Backup docker volumes."
  echo "Syntax: docker-volume-backup [-v|-a|-o|-c|help]"
  echo "Example: docker-volume-backup -v postgres_data01 -o /tmp -c postgres01"
  echo "options:"
  echo "  -v    Comma-separated list of volume names."
  echo "  -a    Backup all volumes."
  echo "  -o    Output directory. Defaults to '/var/tmp'"
  echo "  -c    Docker container name."
  echo "  help  Show docker-volume-backup manual."
  echo
}

# Show help and exit
if [[ $1 == 'help' ]]; then
    Help
    exit
fi

# Process params
while getopts ":a :c: :v: :o:" opt; do
  case $opt in
    a) ALL='true'
    ;;
    c) CONTAINER="$OPTARG"
    ;;
    v) VOLUMES="$OPTARG"
    ;;
    o) DIR="$OPTARG"
    ;;
    \?) echo "Invalid option -$OPTARG" >&2
    Help
    exit;;
  esac
done

# Fallback to environment vars and default values
: ${DIR:='/var/tmp'}

# Verify variables
[[ -z "$VOLUMES" ]] && [[ -z "$ALL" ]] && { echo "Parameter -v or -a|volumes or all must be set" ; exit 1; }
[[ -z "$DIR" ]] && { echo "Parameter -d|dir is empty" ; exit 1; }
[[ -z "$CONTAINER" ]] && { echo "Parameter -c|container is empty" ; exit 1; }

if $ALL ; then
  # Get all volumes from docker container
  VOLUME_LIST=($(docker inspect -f '{{ range .Mounts }}{{ .Name }} {{ end }}' $CONTAINER))

  # Concate volume list
  printf -v VOLUMES '%s,' "${VOLUME_LIST[@]}"
  VOLUMES="${VOLUMES%,}"
fi

if [[ ! -z $VOLUMES ]] ; then
  # Split volumes param values
  VOLUME_LIST=($(echo $VOLUMES | tr "," "\n"))
fi

# Create backup folder
mkdir -p ${DIR}/${CONTAINER}

# Cleanup backup folder
rm -rf ${DIR}/${CONTAINER}/*

# Create dump with docker for each database
for VOLUME in "${VOLUME_LIST[@]}"
do
  echo "Run backup for Docker volume $VOLUME"
  docker run --rm -v $VOLUME:/_data  -v ${DIR/./$(PWD)}/${CONTAINER}:/backup ubuntu tar cf /backup/$VOLUME.tar /_data
done

# Notify if backup has finished
echo "The Docker volume backup has finished: ${DIR}/${CONTAINER}/{${VOLUMES}}.tar"
```

To get the latest version of this see the [Ansible Playbooks](https://github.com/Mint-System/Ansible-Playbooks) projects.