---
title: "Manage Vercel DNS records with Ansible"
slug: manage-vercel-dns-records-with-ansible
date: 2021-02-11T11:41:16+01:00
categories:
 - Continuous Delivery
tags:
 - ansible
 - dns
 - vercel
images:
 - /wp-content/uploads/2018/02/Ansible-text-only.png
---

At [Mint System](https://www.mint-system.ch/) we are using Ansible extensively to configure our infrastructure and services. From setting up new hosts up to deploying a customer specific customization for an application, all is managed by Ansible. Only one piece was missing. Until recently we could not update DNS records automatically.

<!--more-->

We use [Vercel](https://vercel.com/) to deploy static sites and manage DNS records. Their well documented API allows you to manage any Vercel resource. Using the Ansible `uri` module, I have created a role that manages these ressources. Lets have a look.

This is the inventory template:

```yml
vercel_token: "{{ vault_vercel_token }}"
vercel_team_id: example-organization
vercel_dns:
  - domain: example.com
    records:
      - { name: www, type: ALIAS, value: www.example.org, state: present }
      - { name: '', type: A, value: 93.184.216.34, state: present }
```

You can configure domains and their DNS records.

This inventory is used by the `vercel-dns` task. I will now step through the task file and explain how DNS records are updated.

**roles/vercel/tasks/vercel-dns.yml**

```yml
- name: Get all vercel domains
  uri:
    url: "{{ vercel_api_url }}/v4/domains?teamId={{ vercel_team_id }}"
    headers:
      Content-Type: "application/json"
      Authorization: "Bearer {{ vercel_token }}"
    return_content: yes
  register: vercel_domains

- name: Set domains exist fact
  set_fact:
    domains_exist: "{{ vercel_domains.json | json_query('domains[*].name') }}"

- name: Fail if domain is not managed by vercel
  fail:
    msg: Domain is not managed by vercel
  when: item.domain not in domains_exist
  loop: "{{ vercel_dns }}"

```

First all managed domains are retrieved from Vercel. The second task checks wether the domain that is in the inventory is also setup in Vercel. If this is not the case, the execution will fail.

```yml
- name: Get all vercel dns records
  uri:
    url: "{{ vercel_api_url }}/v4/domains/{{ item }}/records?teamId={{ vercel_team_id }}"
    headers:
      Content-Type: "application/json"
      Authorization: "Bearer {{ vercel_token }}"
    return_content: yes
  loop: "{{ domains_exist }}"
  register: vercel_dns_records

- name: Set vercel dns simple fact
  set_fact:
    record: "{{ item.1.name }}.{{ item.0.item }}-{{ item.1.type }}"
  with_subelements:
    - "{{ vercel_dns_records.results }}"
    - json.records
  register: vercel_dns_simple

- name: Make a simple list
  set_fact:
    vercel_dns_simple: "{{ vercel_dns_simple.results | map(attribute='ansible_facts.record') | list }}"

- name: Ensure DNS entry exists
  uri:
    url: "{{ vercel_api_url }}/v2/domains/{{ item.0.domain }}/records?teamId={{ vercel_team_id }}"
    method: POST
    headers:
      Content-Type: "application/json"
      Authorization: "Bearer {{ vercel_token }}"
    body: "{{ item.1 | to_json }}"
  with_subelements:
    - "{{ vercel_dns }}"
    - records
  when: ((item.1.name + "." + item.0.domain + "-" + item.1.type) not in vercel_dns_simple) and (item.1.state == 'present')
  register: response
  changed_when: response.status == 200
```

The next four tasks retrieve all DNS records from Vercel for each configured domain. In order to compare the inventory list and the Vercel list, the Vercel list is flattened. Ansible cannot compare objects easily, but comparing strings is fine. In the last step the task checks whether the DNS record exists and if not creates one.

```yml
- name: Set vercel dns absent fact
  set_fact:
    record: "{{ item.1.name }}.{{ item.0.domain }}-{{ item.1.type }}"
  with_subelements:
    - "{{ vercel_dns }}"
    - records
  when: item.1.state == 'absent'
  register: vercel_dns_absent

- name: Make a simple list
  set_fact:
    vercel_dns_absent: "{{ vercel_dns_absent.results | selectattr('ansible_facts.record','defined') | map(attribute='ansible_facts.record') | list }}"

- name: Ensure DNS entries to be removed
  uri:
    url: "{{ vercel_api_url }}/v2/domains/{{ item.0.item }}/records/{{ item.1.id }}?teamId={{ vercel_team_id }}"
    method: DELETE
    headers:
      Content-Type: "application/json"
      Authorization: "Bearer {{ vercel_token }}"
  with_subelements:
    - "{{ vercel_dns_records.results }}"
    - json.records
  when: (item.1.name + "." + item.0.item + "-" + item.1.type) in vercel_dns_absent
  register: response
  changed_when: response.status == 200
```

The last three tasks will remove a DNS record, if it has the status `absent`.  First a simple list is created that contains all records with the absent status. Then foreach Vercel record it will check if it is in the absent list. If so the task will remove the record from Vercel.

As expected the Ansible tasks are idempotent, they will return an ok if nothing has changed. Further it is ensured that DNS records which are configured in Vercel only are not deleted or updated by acccident.

To get the lastes version of the script, have a look here: <https://github.com/Mint-System/Ansible-Playbooks/blob/master/roles/vercel/tasks/vercel-dns.yml>.