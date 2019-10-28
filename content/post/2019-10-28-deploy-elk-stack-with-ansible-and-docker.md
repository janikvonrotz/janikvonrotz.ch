---
title: "Deploy ELK stack with Ansible and Docker"
slug: deploy-elk-stack-with-ansible-and-docker
date: 2019-10-28T17:18:48+01:00
categories:
  - Continiuous delivery
tags:
 - ansible
 - docker
 - elastic search
 - logstash
 - elk
 - stack
 - kibana
 - deployment
images:
 - /images/shipping container.jpg
---

This post was first published at [Abilium - Blog](https://abilium.com/blog/).

Recently, we decided to setup a new monitoring service. We opted for the [ELK stack](https://www.elastic.co/de/what-is/elk-stack). The ELK stack constists of three products:
<!--more-->

1. **Elasticsearch** - A powerful and flexible search index.
2. **Logstash** - Log ingester, filter and forwarder.
3. **Kibana** - Dashboard for data visualization.

There exist multiple distros of the ELK stack. One of them is the [Open Distro for Elasticsearch](https://opendistro.github.io/for-elasticsearch/) by Amazon and another one is the [Elastic Stack](https://www.elastic.co/de/products/elastic-stack) by Elastic.

We decided to use the Elastic Stack as they provide a collection of Beats in addition. Beats are services that ship all kinds of data (Log, Metrics, Performance, Uptime) to Logstash. It makes it much easier to actually collect data of your services and forward them to Logstash.

At Abilium GmbH Docker and Kubernetes are the default way to run applications. Most times we use Jenkings and Docker Compose to build, test and deploy an application release. But we were not quite happy with docker compose as it does not support a meaningful way to configure the host system. Thus we decided use a very popular automation and configuration management tool: [Ansible](https://www.ansible.com/).

In the following paragraph we will show how you can deploy the Elastic Stack using Ansible and Docker.

## Requirements

It's assumed that you already know how to handle Ansible and are familiar with the related terms. Moreover, we assume that you know the basic idea of Docker and how it interacts with Ansible.

Regarding system requirements for Docker checkout the first section of [Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/master/docker.html)

## Setup

Ansible in its plain form is a hierarchy of `.yml` files which tell what task should performed on which machine. There is no final layout when it comes to structuring these files. To get a grasp of what an Ansible project might look like, we printed the folder tree of our example project:


```bash
├── ansible.cfg # Global ansible configuration
├── elk-clean.yml # Cleanup playbook
├── elk.yml # Elastic Stack playbook
├── inventory # Inventory folder
│   ├── group_vars # Folder that contains variables grouped by environment
│   │   ├── all # Variables which apply to every environment
│   │   │   ├── vars.yml # Global configuration
│   │   │   └── vault.yml # Ansible Vault secrets
│   │   └── prod # Production environment
│   │       └── vars.yml # Environment specific folder
│   └── hosts.yml # List of hosts grouped by environment
└── roles # Roles are Ansible modules
    ├── clean # Cleanup role that removes all configurations
    │   └── tasks
    │       └── main.yml # Tasks to cleanup containers and related config
    ├── elasticsearch # The Elasticsearch role
    │   └── tasks
    │       └── main.yml  # Tasks to deploy the Elasticsearch Docker container
    ├── kibana # The Kibana role
    │   └── tasks
    │       └── main.yml # Tasks to deploy the Kibana Docker container
    ├── logstash # The Logstash role
    │   ├── handlers
    │   │   └── main.yml # Tasks which are called by other tasks via notifier
    │   ├── tasks
    │   │   └── main.yml # Tasks to deploy the Logstash Docker container
    │   └── templates # Logstash configuration files with Ansible variables
    │       ├── beats.conf
    │       └── syslog.conf
    └── nginx # The nginx module to expose the Kibana dashboard
        ├── handlers
        │   └── main.yml
        ├── tasks
        │   └── main.yml
        └── templates
            ├── nginx-certbot.conf
            └── nginx-ssl.conf
```

You can replicate this folder tree or checkout the single files in the follow-up sections.

## Elasticsearch

This Ansible task file deploys the Elasticsearch Docker container to the target host.

**roles/elasticsearch/tasks/main.yml**

```yml
- name: Create elastic search volume
  docker_volume:
    name: esdata
    driver: local

- name: Start elastic search container
  docker_container:
    name: "{{ elasticsearch_hostname }}"
    image: "{{ elasticsearch_image }}"
    env:
      discovery.type: "single-node"
      ES_JAVA_OPTS: "-Xms512m -Xmx512m"
      xpack.security.enabled: "true"
      xpack.monitoring.collection.enabled: "true"
    volumes:
    - "esdata:/usr/share/elasticsearch/data"
    ulimits:
    - nofile:65535:65535
    networks:
    - name: "{{ network_name }}"
    state: started
    log_driver: "{{ log_driver }}"
    log_options:
      max-size: "{{ log_max_size }}"
      max-file: "{{ log_max_file }}"

- debug:
    msg: >
      Users have to be configured manually. Enable and set the password for the default users with:
      `docker exec -it {{ elasticsearch_hostname }} /bin/bash -c "elasticsearch-setup-passwords auto"`
      Then edit the `vaul.yml` file and copy the password values to the expected variable.
```

## Kibana

Kibana is configured with env variables only.

**roles/kibana/tasks/main.yml**

```yml
- name: Start kibana container
  docker_container:
    name: "{{ kibana_hostname }}"
    image: "{{ kibana_image }}"
    env:
      servername: "{{ server_name }}"
      ELASTICSEARCH_HOSTS: "http://{{ elasticsearch_hostname }}:9200"
      ELASTICSEARCH_USERNAME: "kibana"
      ELASTICSEARCH_PASSWORD: "{{ kibana_password }}"
    networks:
    - name: "{{ network_name }}"
    log_driver: "{{ log_driver }}"
    log_options:
      max-size: "{{ log_max_size }}"
      max-file: "{{ log_max_file }}"
```

## Logstash

Logstash processes data with pipelines. Each pipeline is configured in a `.conf` file. These files are deployed with Ansible as well. Everytime a file change occurs Ansible will restart the Logstash container.

**roles/logstash/tasks/main.yml**

```yml
- name: Ensure logstash pipeline conf dir exists
  file:
    path: "{{ logstash_conf_dir }}/pipeline"
    state: directory

- name: Copy logstash pipeline conf
  template: src={{ item.src }} dest={{ item.dest }}
  with_items:
    - { src: 'templates/syslog.conf', dest: '/{{ logstash_conf_dir }}/pipeline/syslog.conf' }
    - { src: 'templates/beats.conf', dest: '/{{ logstash_conf_dir }}/pipeline/beats.conf' }
  notify: Restart logstash container

- name: Start logstash container
  docker_container:
    name: "{{ logstash_hostname }}"
    image: "{{ logstash_image }}"
    env:
      XPACK_MONITORING_ELASTICSEARCH_HOSTS: "http://{{ elasticsearch_hostname }}:9200"
      XPACK_MONITORING_ENABLED: "true"
      XPACK_MONITORING_ELASTICSEARCH_USERNAME: "elastic"
      XPACK_MONITORING_ELASTICSEARCH_PASSWORD: "{{ elastic_password }}"
      PATH_CONFIG: ""
    ports:
    - 5000:5000
    - 5044:5044
    volumes:
    - "{{ logstash_conf_dir }}/pipeline:/usr/share/logstash/pipeline:ro"
    networks:
    - name: "{{ network_name }}"
    log_driver: "{{ log_driver }}"
    log_options:
      max-size: "{{ log_max_size }}"
      max-file: "{{ log_max_file }}"
```

**roles/logstash/handlers/main.yml**

Handlers are notified by tasks and will be processed at the end of a role deployment.

```yml
- name: Restart logstash container
  docker_container:
    name: "{{ logstash_hostname }}"
    restart: true
```

**roles/logstash/templates/beats.conf**

The beats pipeline forwards messages sent by Beats services to its appropriate index.

```conf
input { stdin { } }

input {
  beats {
    port => 5044
  }
}

output {
  if [@metadata][beat] in ["heartbeat", "metricbeat", "filebeat"] {
    elasticsearch { 
      hosts => ["http://{{ elasticsearch_hostname }}:9200"]
      user => "elastic"
      password => "{{ elastic_password }}"
      index => "%{[@metadata][beat]}-%{[@metadata][version]}"
    }
  }
}
```

**roles/logstash/templates/syslog.conf**

The syslog pipeline processes syslog messages using a grok filter.

```conf
input { stdin { } }

input {
  tcp {
    port => 5000
    type => syslog
  }
  udp {
    port => 5000
    type => syslog
  }
}

filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOG5424PRI}%{NONNEGINT:syslog5424_ver} +(?:%{TIMESTAMP_ISO8601:syslog5424_ts}|-) +(?:%{HOSTNAME:syslog5424_host}|-) +(?:%{NOTSPACE:syslog5424_app}|-) +(?:%{NOTSPACE:syslog5424_proc}|-) +(?:%{WORD:syslog5424_msgid}|-) +(?:%{SYSLOG5424SD:syslog5424_sd}|-|) +%{GREEDYDATA:syslog5424_msg}" }
    }
    syslog_pri { }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
    if !("_grokparsefailure" in [tags]) {
      mutate {
        replace => [ "@source_host", "%{syslog_hostname}" ]
        replace => [ "@message", "%{syslog_message}" ]
      }
    }
    mutate {
      remove_field => [ "syslog_hostname", "syslog_message", "syslog_timestamp" ]
    }
  }
}

output {
  if [type] == "syslog" {
    elasticsearch { 
      hosts => ["http://{{ elasticsearch_hostname }}:9200"]
      user => "elastic"
      password => "{{ elastic_password }}"
      index => "syslog-%{+YYYY.MM.dd}"
    }
  }
}
```

## Nginx

It is easier to setup an Nginx proxy that secures access to the Kibana dashboard than configuring keymaterial for Kibana and setup direct access.

**roles/nginx/tasks/main.yml**

First this role checks if LetsEncrypt certificates have been generated for the configured domain. If this is not the case it will setup an Nginx instance whose only purpose is to complete the ACME challenge initialized by the Certbot command. Once the certificates have been generated a ssl secured Nginx instance will be deployed.

```yml
- name: Ensure nginx conf dir exists
  file:
    path: "{{ nginx_conf_dir }}"
    state: directory

- name: Check if cert files exist
  stat:
    path: "{{ certbot_conf_dir }}/live/{{ server_name }}"
  register: certbot_certs

- name: Copy nginx certbot conf
  template:
    src: templates/nginx-certbot.conf
    dest: "{{ nginx_conf_dir }}/{{ server_name }}.conf"
  when: not certbot_certs.stat.exists

- name: Start nginx container
  docker_container:
    name: ng01
    image: "{{ nginx_image }}"
    ports:
    - 80:80
    volumes:
    - "{{ nginx_conf_dir }}/:/etc/nginx/conf.d/:ro"
    - "{{ certbot_conf_dir }}/:/etc/letsencrypt/"
    - "{{ certbot_conf_dir }}/www/:/var/www/certbot/"
    log_driver: "{{ log_driver }}"
    log_options:
      max-size: "{{ log_max_size }}"
      max-file: "{{ log_max_file }}"
  when: not certbot_certs.stat.exists

- name: Wait for nginx container
  pause:
    seconds: 3
  when: not certbot_certs.stat.exists

- name: Issue certificate with certbot command
  docker_container:
    name: "{{ certbot_hostname }}"
    image: "{{ certbot_image }}"
    volumes:
    - "{{ certbot_conf_dir }}/:/etc/letsencrypt/"
    - "{{ certbot_conf_dir }}/www/:/var/www/certbot/"
    command: certonly --webroot --email {{ certbot_email }} --agree-tos --webroot-path=/var/www/certbot/ -d {{ server_name }}
  when: not certbot_certs.stat.exists

- name: Wait for certificate request
  pause:
    seconds: 3
  when: not certbot_certs.stat.exists

- name: Copy nginx ssl conf
  template:
    src: templates/nginx-ssl.conf
    dest: "{{ nginx_conf_dir }}/{{ server_name }}.conf"
  notify: Restart nginx container

- name: Copy nginx ssl param conf
  copy:
    src: "{{ item }}"
    dest: "{{ certbot_conf_dir }}/"
  with_items:
    - files/options-ssl-nginx.conf
    - files/ssl-dhparams.pem

- name: Start nginx container
  docker_container:
    name: ng01
    image: nginx:1.15-alpine
    ports:
    - 80:80
    - 443:443
    - 9200:9200
    volumes:
    - "{{ nginx_conf_dir }}/:/etc/nginx/conf.d/:ro"
    - "{{ certbot_conf_dir }}/:/etc/letsencrypt/"
    - "{{ certbot_conf_dir }}/www/:/var/www/certbot/"
    networks:
    - name: "{{ network_name }}"
    log_driver: "{{ log_driver }}"
    log_options:
      max-size: "{{ log_max_size }}"
      max-file: "{{ log_max_file }}"
```

**roles/nginx/handlers/main.yml**

The Nginx handler for config updates.

```yml
- name: Restart nginx container
  docker_container:
    name: ng01
    restart: true
```

**roles/nginx/templates/nginx-certbot.conf**

The ACME challenge Nginx config.

```conf
server {
    listen 80;
    server_name {{ server_name }};

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    } 
}
```

**roles/nginx/templates/nginx-ssl.conf**

The ssl Nginx config that secures access to the Elastic Search API and Kibana dashboard.

```conf
server {
    listen 80;
    server_name {{ server_name }};    
    
    location / {
        return 301 https://$host$request_uri;
    }

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    } 
}

# Kibana Dashboard
server {
    listen 443 ssl;
    server_name {{ server_name }};

    ssl_certificate /etc/letsencrypt/live/{{ server_name }}/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/{{ server_name }}/privkey.pem;
    
    location / {
        proxy_pass http://{{ kibana_hostname }}:5601;
    }
}

# Elastic Search
server {
    listen 9200 ssl;
    server_name {{ server_name }};

    ssl_certificate /etc/letsencrypt/live/{{ server_name }}/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/{{ server_name }}/privkey.pem;

    location / {
        proxy_pass http://{{ elasticsearch_hostname }}:9200;
    }
}
```

Note that cipher suite definitions have been removed from the example. Recommended cipher suites must be obtained by a security consultancy.

## Inventory

Ansible works against multiple environments. Therefore configurations and code must be separated. Whereas roles are the code, configurations are the inventory.

**inventory/hosts.yml**

```
all:
  hosts:
  children:
    prod:
      hosts: monitoring.example.com
    int:
      hosts: monitoring-int.example.com
```

**inventory/group_vars/all/vars.yml**

These variables are set for all environments.

```yml
log_driver: "json-file"
log_max_size: "10m"
log_max_file: "3"
```

They configure the log rotation for the Docker daemon.

**inventory/group_vars/all/vault.yml**

The vault file stores secrets. It is set for all environments as well. Run the this command to edit the file:

`ansible-vault edit inventory/group_vars/all/vault.yml`

```yml
vault_elastic_password: password
vault_kibana_password: password
```

These passwords must be updated after the first depoyment.

**inventory/group_vars/prod/vars.yml**

If you run the Ansible deployment it will use the prod environment definitions by default. The prod environment has been configured with these variables:

```yml
server_name: monitoring.example.com
network_name: esnet

nginx_image: nginx:1.15-alpine
nginx_hostname: ng01
nginx_conf_dir: /usr/share/nginx

elasticsearch_image: docker.elastic.co/elasticsearch/elasticsearch:7.4.0
elasticsearch_hostname: es01
elasticsearch_conf_dir: /usr/share/elasticsearch

kibana_image: docker.elastic.co/kibana/kibana:7.3.2
kibana_hostname: ki01

logstash_image: docker.elastic.co/logstash/logstash:7.4.0
logstash_hostname: lo01
logstash_conf_dir: /usr/share/logstash

certbot_image: certbot/certbot
certbot_hostname: cb01
certbot_conf_dir: /usr/share/certbot
certbot_email: info@example.com

elastic_password: "{{ vault_elastic_password }}"
kibana_password: "{{ vault_kibana_password }}"
```

The hostname is used inside the Docker network.

## Playbooks

We are getting closer to the actual Ansible deployment. Ansible playbooks connect hosts, environments and roles. They describe which role must be installed on which host.

**elk.yml**

This playbook installs the described roles. Note that you can use tags to filter roles for deployments.

```yml
---
- hosts: "{{ ehosts | default('prod') }}"
  become: true
  roles:
  - { role: elasticsearch, tags: ["elasticsearch"] }
  - { role: kibana, tags: ["kibana"] }
  - { role: logstash, tags: ["logstash"] }
  - { role: nginx, tags: ["nginx"] }
```

**elk-clean.yml**

The cleanup process has been separated from the installation. Cleaning up installed software requires a different order, therefore the cleanup tasks received an exclusive role.

```yml
- hosts: "{{ ehosts | default('prod') }}"
  become: true
  roles:
  - clean
```

## Deployment

You have reached the most important section. Here we show what commands can be used to deploy the ELK stack with Ansible and Docker.

Deploy the ELK stack.

`ansible-playbook -i inventory elk.yml`

Deploy the Logstash role only.

`ansible-playbook -i inventory elk.yml --tags logstash`

Deploy the Nginx role to localhost.

`ansible-playbook -i inventory elk.yml --tags nginx --extra-vars "ehosts=local"`

Deploy the Nginx role to localhost with a specified user.

`ansible-playbook -i inventory elk.yml --tags nginx --extra-vars "ehosts=local" -u username`

Clean the ELK stack.

`ansible-playbook -i inventory elk-clean.yml`

Clean the Logstash role only.

`ansible-playbook -i inventory elk-clean.yml --tags logstash`

## Demo

Let's see what a deployment looks like in action:

<script id="asciicast-8rCcWp8dSaj2OpbCp3JIeb8pq" src="https://asciinema.org/a/8rCcWp8dSaj2OpbCp3JIeb8pq.js" async></script>

That's it! I hope you were able to follow our solution. If not let us know in the comment section.

## Notes

As you might noticed there are parts of the Ansible project that have not been solved nicely. F.g. The default users have to be configured manually. We wanna address this points and show how to resolve them if there would be more ressources for engineering.

Problem: Manual setup of default users.  
Solution: Create and set the password of the elastic and kibana user automatically with http requests.

Problem: Beat and syslog input services communication is not encrypted. 
Solution: Encrypt connection to Logstash beat and syslog input with LetsEncrypt certificates.

Problem: Document role variables.
Solution: Generate a documentation based on the inventory vars and comments.