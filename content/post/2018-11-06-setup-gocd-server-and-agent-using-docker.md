---
title: "Setup GoCD environment using docker"
slug: 2018-11-06-setup-gocd-environment-using-docker
date: 2018-11-06T12:11:01+02:00
categories:
 - Software Development
tags:
 - continuous delivery
 - gocd
 - introduction
 - docker
 - tutorial
images:
 - /images/gocd-logo.PNG
---

At [AdNovum](https://www.adnovum.ch/) we are running [GoCD](https://www.gocd.org/) for automating the continuous delivery (CD) of our software products. GoCD has become the de facto standard for running CD pipelines.

In this post I will give you a short introduction into GoCD using [docker](https://www.docker.com/). We are going to setup a GoCD environment consisting of a server, an agent and deployable material. Further we are going to configure a deployment pipeline and build an node.js application.
<!--more-->

## GoCD environment

The GoCD server provides an interface for configuring pipelines, environments and agents. The agent does the actual work if a deployment pipeline is triggered. Workload is managed by the GoCD server and shared among multiple agents.

In order setup our GoCD environment you need to install [docker](https://www.docker.com/get-started).

If docker is available, start by downloading the official GoCD server image.

`docker pull gocd/gocd-server:v18.10.0`

Next start a new GoCD server container.

`docker run -d --name gocd-server -p8153:8153 -p8154:8154 gocd/gocd-server:v18.10.0`

In your browser open [https://localhost:8154](https://localhost:8154). In this interface you can manage the GoCD server.

In order to run a pipeline we need a GoCD agent.

Pull the official GoCD alpine image.

`docker pull gocd/gocd-agent-alpine-3.8:v18.10.0`

Run the command below to start a new GoCD agent container.

`docker run -itd  --name gocd-agent -e CI=true -e GO_SERVER_URL=https://$(docker inspect --format='{{(index (index .NetworkSettings.IPAddress))}}' gocd-server):8154/go gocd/gocd-agent-alpine-3.8:v18.10.0`

Make sure that server and agent have successfully been started.

`docker ps`

The agent is registering itself ùsing the `GO_SERVER_URL` variable and should be listed in the management interface  [https://localhost:8154/go/agents#!/agentState/asc/](https://localhost:8154/go/agents#!/agentState/asc/).

If the GoCD agent entered the state pending, select it and click on *Enable*.

# Agent configuration

Materials are the input for a build pipeline in GoCD. A material can be anything, an svn repo, folder, file or as in most cases a git repo. We are going to use the [gocd-node-example](https://github.com/janikvonrotz/gocd-node-material) app as our material.

Building the application requires two binaries:

* yarn
* node

These binaries are not installed on the image by default, so we need to update the image.

Log into the `gocd-agent` container.

`docker exec -it gocd-agent bash`

Install the binaries using the alpine package manager.

`apk add --no-cache nodejs yarn`

Logout and update the container image.

```sh
docker stop gocd-agent
docker commit gocd-agent gocd-node-agent
docker start gocd-agent
docker images
```

The commited image should be listed. If you spin up a GoCD agent again, use the new image.

# Pipeline configuration

Now we are ready to configure and run a GoCD pipeline.

In the management interface create a new pipeline with the name `gocd-node-example`.

Select *Git* as material type and enter this url: `https://github.com/janikvonrotz/gocd-node-material.git`

A pipeline passes a material and its resulting artifacts through multiple stages. Each stage performs a build procedure or performs checks on the artifact. Here is an example of a 4-stage pipeline:

1. build
2. test
3. analyze
4. publish

We are going to have configure a build stage with two tasks and a test stage with one task.

Name the current stage *build* and the job name *build*.

Select *More...* as task type and enter `/bin/bash` in the command field.

In the argument field add:

```
-c
yarn
```

Finish the wizard and open the tab *Stages*.

Select the *build* stage, then the *build* job and add a new task.

task type: *More...*  
command: `/bin-bash`  
argument:

```
-c
yarn build
```

Add one more stage with the following details:

stage name: *test*  
job name: *test*  
task type: *More...*  
command: `/bin-bash`  
argument:

```
-c
yarn test
```

Once all stages have been created, unpause and run the pipeline.

![gocd pipeline in progress](/images/gocd pipeline in progress.PNG)

GoCD will poll the master branch for changes. Whenver a change is committed GoCD will trigger the pipeline.

## Troubleshooting

**proxy settings**

In case the docker host runs behind a proxy and GoCD cannot connect to the git repository, make sure to set the git config proxy withing the docker container.

```
docker exec -it _CONTAINER_NAME_ bash
su go
git config --global http.proxy https://_PROXY_HOST_:_PROXY_PORT_
git config --global https.proxy HTTP_PROXY=http://_PROXY_HOST_:_PROXY_PORT_
```

For other outbound connection you can also pass the proxy env vars via docker.

`docker run -e HTTPS_PROXY=https://_PROXY_HOST_:_PROXY_PORT_ -e HTTP_PROXY=http://_PROXY_HOST_:_PROXY_PORT_ ...`

**github connection refused**

If you cannot connect with the github repository and get this fatal error:

`fatal: unable to access 'https://github.com/janikvonrotz/gocd-node-material.git/': Failed to connect to github.com port 443: Connection refused`

It is most likely a proxy related problem.

**wrong version number**

Assuming the proxy env vars have been and you get this error:

`STDERR: fatal: unable to access 'https://github.com/janikvonrotz/gocd-node-material.git/': error:1400410B:SSL routines:CONNECT_CR_SRVR_HELLO:wrong version number`

Make sure to set the git proxy config for the go user:

```
docker exec -it _CONTAINER_NAME_ bash
su go
git config --global http.proxy https://_PROXY_HOST_:_PROXY_PORT_
git config --global https.proxy HTTP_PROXY=http://_PROXY_HOST_:_PROXY_PORT_
```

**GoCD agent pending**

If the GoCD agent entered the state pending, select it and click on *Enable*.

**yarn install job fails**

Assuming the following error occurs during the install stage:

```
...
[1/4] Resolving packages...
info If you think this is a bug, please open a bug report with the information provided in "/godata/pipelines/build/yarn-error.log".
...
```

You need to set the yarn proxy settings in the `gocd-agent` container.

```
yarn config set proxy http://_PROXY_HOST_:_PROXY_PORT_
yarn config set https-proxy https://_PROXY_HOST_:_PROXY_PORT_
```