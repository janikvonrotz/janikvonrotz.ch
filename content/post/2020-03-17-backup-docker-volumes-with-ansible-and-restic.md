---
title: "Backup Docker volumes with Ansible and restic"
slug: backup-docker-volumes-with-ansible-and-restic
date: 2020-03-16T17:43:01+01:00
categories:
 - DevOps
tags:
 - backup
 - ansible
 - restic
images:
 - "/images/vault boxes.jpg"
---

In a new assignment I'm in charge of the infrastructure for a new startup. I was given a blank canvas and decided to use Ansible and Docker from the start. Therefore I've setup an Ansible project containing various roles and deployment scenarios. Have a look here for details: [https://github.com/Mint-System/Ansible-Playbooks](https://github.com/Mint-System/Ansible-Playbooks). To put it simply, this project deploys open source web application as Docker containers on a target system. Currently, I am adding new features and polishing existing ones. An important role that is still missing is the backup. Having a robust and reliable backup and recovery system is key. While developing the backup system I had a few key points in mind:
<!--more-->

* Backup Docker volumes
* Backup location is remote
* No friction between backup and recovery
* Backup tool must manage rotation policies
* Docker host stores backups
* Support for multiple backup clients and servers
* Encrypted backups
* Simple configuration

## Solution

In the past I have used [duplicity](http://duplicity.nongnu.org/) for my backups. I remember it as quite handy for managing backups, but complicated regarding encryption. While doing more research (aka googling *duplicity alternatives*), I found [restic](https://restic.net/). It seems restic has become the standard for remote backups. Folks from [camptocamp](https://www.camptocamp.com/) use it to backup their kubernetes cluster using the restic based backup tool [bivac](https://github.com/camptocamp/bivac). Obviously, restic has been proven to be a worthy candidate.

## Files

First I want to give you a short intro to the Ansible project. I have isolated the inventory and roles of the restic client and server deployment.

These are the relevant Ansible project files:

```bash
.
├── backup.yml
├── inventories/
│   └── backup/ # name of the inventory
│       ├── group_vars/
│       │   ├── all.yml
│       │   ├── client/
│       │   │   ├── vars.yml
│       │   │   └── vault.yml # Use ansible-vault to create this file
│       │   └── server/
│       │       ├── vars.yml
│       │       └── vault.yml  # Use ansible-vault to create this file
│       ├── host_vars/
│       │   ├── client.example.com.yml
│       │   └── server.example.com.yml
│       └── hosts.yml
└── roles/
    ├── restic-client/
    │   └── tasks/
    │   │   ├── main.yml
    │       └── install.yml
    └── restic-server/
        └── tasks/
            ├── main.yml
            └── main.yml
```

## Inventory

Let's have a look at the inventory.

The `all.yml` file contains configs which is applied to all hosts of the backup inventory.

**inventories/backup/group_vars/all.yml**

```yml
docker_network_name: example.com
docker_log_driver: "json-file"
docker_log_max_size: "10m"
docker_log_max_file: "3"

ansible_python_interpreter: /usr/bin/python3
```

For every inventory group, in this case client and server, there is a folder and a vars config file.

**inventories/backup/group_vars/client/vars.yml**

```yml
restic_client_package: "restic=0.8.3+ds-1"
restic_client_user: admin
restic_client_password: "{{ vault_restic_client_password }}"
restic_repo: server.example.com:8080/backup
restic_repo_password: "{{ vault_restic_repo_password }}"
```

Every variable that is prefixed with *vault_* is defined in the `vault.yml` file.

**inventories/backup/group_vars/server/vars.yml**

```yml
# https://hub.docker.com/r/restic/rest-server
restic_server_image: restic/rest-server:0.9.7
restic_server_user: admin
restic_server_password: "{{ vault_restic_server_password }}"
restic_server_port: 8080
```

For host specific configurations there is a file in the `host_vars` folder.

**inventories/backup/host_vars/client.example.com.yml**

```yml
restic_backup_sets:
 - id: "odoo volume"
   type: docker-volume
   volume: odoo_data01
   tags:
    - odoo01
    - postgres01
   hour: "1"
   minute: "0"
 - id: "postgres volume"
   type: docker-volume
   volume: postgres_data01
   tags:
    - odoo01
    - postgres01
   hour: "1"
   minute: "0"
restic_backup_rotation:
  daily: 7
  weekly: 4
  monthly: 1
```

Each client has a set of backup jobs.

**inventories/backup/host_vars/server.example.com.yml**

```yml
restic_server_hostname: restic01
restic_server_backup_dir: /path/to/mount
```

The server mounts the backupfolder via bind mount.

## Roles

Next we will have a look at the Ansible roles.

### Restic server

The restic server receives and stores backup files.

**roles/restic-server/tasks/main.yml**

```yml
- name: Include install tasks
  include_tasks: install.yml
  when: restic_server_image is defined
```

If a restic server docker image is defined, the install tasks are included.

**roles/restic-server/tasks/install.yml**

```yml
- name: Install htpasswd module
  apt: 
    name: python3-passlib
    state: latest

- name: Configure user access for restic server
  htpasswd:
    path: "{{ restic_server_backup_dir }}/.htpasswd"
    name: "{{ restic_server_user }}"
    crypt_scheme: ldap_sha1
    password: "{{ restic_server_password }}"

- name: Start restic server container
  docker_container:
    name: "{{ restic_server_hostname }}"
    image: "{{ restic_server_image }}"
    volumes:
      - "{{ restic_server_backup_dir }}:/data"
    ports:
      - "{{ restic_server_port }}:8000"
    networks:
      - name: "{{ docker_network_name }}"
    log_driver: "{{ docker_log_driver }}"
    log_options:
      max-size: "{{ docker_log_max_size }}"
      max-file: "{{ docker_log_max_file }}"
```

The install tasks creates a `.htpasswd` file in the mounted directory and then starts a restic server container with configurations applied from the inventory.

As you can see the http communication is not encrypted. However, this is not a problem as restic encrypts its payload.

### Restic client

The restic client runs backups and uses the server as storage.

**roles/restic-client/tasks/main.yml**

```yml
- name: Include install tasks
  include_tasks: install.yml
  when: restic_client_package is defined
```

Now comes the most interesting part. The `install.yml` of the restic client does a few things:

* Install the restic package
* Check if backup repo has been initialized and if not does so
* Ensure the repo url and password are set as global environment variables
* Register the backup jobs
* Register the backup rotation job

**roles/restic-client/tasks/install.yml**

```yml
- name: Install restic
  apt:
    name: "{{ restic_client_package }}"

- name: Check if repo is initialized
  shell: restic snapshots
  environment:
    RESTIC_PASSWORD: "{{ restic_repo_password }}"
    RESTIC_REPOSITORY: "rest:http://{{ restic_client_user }}:{{ restic_client_password }}@{{ restic_repo }}"
  ignore_errors: yes
  changed_when: false
  register: repo_initalized

- name: Init restic repository
  shell: restic init
  environment:
    RESTIC_PASSWORD: "{{ restic_repo_password }}"
    RESTIC_REPOSITORY: "rest:http://{{ restic_client_user }}:{{ restic_client_password }}@{{ restic_repo }}"
  when: repo_initalized.failed

- name: Ensure restic environment vars exists
  lineinfile:
    dest: "/etc/environment"
    state: present
    regexp: "^{{ item.key }}="
    line: "{{ item.key }}={{ item.value}}"
  loop:
    - key: RESTIC_PASSWORD
      value: "{{ restic_repo_password }}"
    - key: RESTIC_REPOSITORY
      value: "rest:http://{{ restic_client_user }}:{{ restic_client_password }}@{{ restic_repo }}"

- name: Register docker volume backup jobs
  cron:
    name: "Backup job {{ item.id }}"
    hour: "{{ item.hour }}"
    minute: "{{ item.minute }}"
    job: ". /etc/environment; restic backup /var/lib/docker/volumes/{{ item.volume }}/_data/ --tag {{ item.tags | join(' --tag ')}}"
  loop: "{{ restic_backup_sets }}"
  when: item.type == "docker-volume"

- name: Register backup rotation job
  cron:
    name: "Backup rotation job"
    hour: "23"
    minute: "0"
    job: ". /etc/environment; restic forget --keep-daily {{ restic_backup_rotation.daily }} --keep-weekly {{ restic_backup_rotation.weekly }} --keep-monthly {{ restic_backup_rotation.monthly }} --prune"
```

For every item in the `restic_backup_sets` var a cron job is configured. The `item.type` property allows you to define and select more backup types.

## Deployment

This is the final section. I will show you how to deploy the roles.

### Playbook

First we need an Ansible playbook.

**backup.yml**

```yml
- hosts: all
  become: true
  roles:
  - role: restic-server
    tags: restic-server
  - role: restic-client
    tags: restic-client
```

This playbook includes the role. Note that we deploy as root on the target machine.

### Installation

If everything has been configured properly, we can install the server:

`ansible-playbook -i inventories/backup backup.yml -l server.example.com`

And the client:

`ansible-playbook -i inventories/backup backup.yml -l client.example.com`

Here is an example output of the client installation:

```bash
MacBuechLuft:ansible-playbooks janikvonrotz$ ansible-playbook -i inventories/backup backup.yml -t restic-client

PLAY [all] ***********************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************************************************
ok: [server.example.com]
ok: [client.example.com]

TASK [restic-client : Include install tasks] *************************************************************************************************************************************************************************************************
skipping: [server.example.com]
included: /Users/janikvonrotz/ansible-playbooks/roles/restic-client/tasks/install.yml for client.example.com

TASK [restic-client : Install restic] ********************************************************************************************************************************************************************************************************
ok: [client.example.com]

TASK [restic-client : Check if repo is initialized] ******************************************************************************************************************************************************************************************
ok: [client.example.com]

TASK [restic-client : Init restic repository] ************************************************************************************************************************************************************************************************
skipping: [client.example.com]

TASK [restic-client : Ensure restic environment vars exists] *********************************************************************************************************************************************************************************
ok: [client.example.com] => (item={'key': 'RESTIC_PASSWORD', 'value': '********'})
ok: [client.example.com] => (item={'key': 'RESTIC_REPOSITORY', 'value': 'rest:http://admin:'********'})@server.example.com:8080/backup'})

TASK [restic-client : Register docker volume backup jobs] ************************************************************************************************************************************************************************************
ok: [client.example.com] => (item={'id': 'odoo volume', 'type': 'docker-volume', 'volume': 'odoo_data01', 'tags': ['odoo', 'odoo02', 'postgres03'], 'hour': '1', 'minute': '0'})
ok: [client.example.com] => (item={'id': 'postgres volume', 'type': 'docker-volume', 'volume': 'postgres_data01', 'tags': ['postgres', 'odoo02', 'postgres03'], 'hour': '1', 'minute': '0'})

TASK [restic-client : Register backup rotation job] ******************************************************************************************************************************************************************************************
ok: [client.example.com]

PLAY RECAP ***********************************************************************************************************************************************************************************************************************************
client.example.com      : ok=7    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
server.example.com     : ok=1    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0  
```

Note that the passwords are not hidden in your output. They might end up on a log aggregation server.

### Manual backups

Remember that we installed the restic package on the target host and defined two important environment variables on the global scope? This helps managing backups without looking up any secrets.

I can easily browse and if necessary restore backups with personal user.

```bash
MacBuechLuft:~ janikvonrotz$ ssh client.example.com
janikvonrotz@hades:~$ restic snapshots
password is correct
ID        Date                 Host        Tags            Directory
----------------------------------------------------------------------
a8f86bf6  2020-03-23 14:07:02  hades       odoo01      ┌── /var/lib/docker/volumes/odoo_data01/_data
                                           postgres01  └── 
c0686e66  2020-03-23 14:07:02  hades       odoo01      ┌── /var/lib/docker/volumes/postgres_data01/_data
                                           postgres01  └── 
----------------------------------------------------------------------
2 snapshots
```

Cool isn't it? If you have any feedback, let me know? In the near future I will add new backup types and share them in the referenced GitHub repo.
