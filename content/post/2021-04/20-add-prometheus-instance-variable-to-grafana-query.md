---
title: "Add Prometheus instance variable to Grafana query"
slug: add-prometheus-instance-variable-to-grafana-query
date: 2021-04-19T18:05:23+02:00
categories:
 - System Engineering
tags:
 - grafana
 - prometheus
images:
 - /images/use-grafana-instance-variable.png
---

Prometheus stores the instance or host name for every metric. If you want to filter your dashboard data based on the instance name you need a Grafana variable.

<!--more-->

Open the Grafana, select your dashboard and navigate to *Settings > Variables*. Create a new variable:

Name: `isntance`\
Typoe: `Query`\
Lable: `Instance`\
Data source: `Prometheus`\
Query: `label_values(instance)`\
Include All option: `true`

Check if the preview returns the instances.

Next open a panel and insert `{instance=~"$instance"}` into your metric like this:

![](/images/add-varaible-to-grafana-query.png)

Now all the panel data will be filtered by whatever is set for the variable.