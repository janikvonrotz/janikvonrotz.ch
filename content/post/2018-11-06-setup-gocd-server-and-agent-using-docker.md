---
title: "Setup GoCD server and agent using docker"
slug: 2018-11-06-setup-gocd-server-and-agent-using-docker
date: 2018-11-05T12:11:01+02:00
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
draft: true
---

At [AdNovum](https://www.adnovum.ch/) we are running [GoCD](https://www.gocd.org/) for automating the continuous delivery (CD) of our software products. GoCD has become the de facto standard for running CD pipelines.

In this post I will give you a short introduction into GoCD using [docker](https://www.docker.com/). We are going to setup a GoCD environment consisting of a server, an agent and deployable material. Further we are going to configure a deployment pipeline and build an node.js application.
<!--more-->

## GoCD environment

The GoCD server provides an interface for configuring pipelines, environments and agents. The agent does the actual work if a deployment pipeline is triggered. Workload is managed by GoCD server and is shared among agents.

In order setup our GoCD environment you need to install [docker](https://www.docker.com/get-started).

If docker is available, start by downloading the official GoCD server image.

`docker pull gocd/gocd-server:v18.10.0`

Next start a new container.

`docker run -d  --name gocd-server -p8153:8153 -p8154:8154 gocd/gocd-server:v18.10.0`

In your browser open [https://localhost:8154](https://localhost:8154). In this interface you manage the GoCD server.

In order to run a pipeline we need a GoCD agent.

Pull the official gocd alpine image.

`docker pull gocd/gocd-agent-alpine-3.8:v18.10.0`

Run the command below to start a new GoCD agent container.

`docker run -itd  --name gocd-agent -e GO_SERVER_URL=https://$(docker inspect --format='{{(index (index .NetworkSettings.IPAddress))}}' gocd-server):8154/go gocd/gocd-agent-alpine-3.8:v18.10.0`

Make sure that server and agent have been successfully started.

docker ps`

The agent is registering itself ùsing the `GO_SERVER_URL` variable and should be listed in the management interface  [https://localhost:8154/go/agents#!/agentState/asc/](https://localhost:8154/go/agents#!/agentState/asc/).

# Agent configuration

Materials are the build pipeline input in GoCD. It can be anything, an svn repo, folder, file or as in most cases a git repo. We are going to use the [gocd-node-example]() app as our material.

Building the application requires two binaries:

* yarn
* node

These binaries are not installed on the image by default, so we need to update the image.

Login to the gocd-agent container.

`docker exec -it gocd-agent bash`

Install the binaries using the alpine package manager.

`apk add --no-cache nodejs yarn`

Logout and update the container image.

```sh
docker commit gocd-agent gocd-node-agent
docker images
```

The commited image should be listed. If you spin up a gocd agent again, use the new image.

# Pipeline configuration

Now we are ready to configure and run a GoCD pipeline. Every pipeline requires materials as input. A material is usually a git repository.

In the management interface create a new pipeline with the name `gocd-node-example`.

Select *Git* as material type and enter this url: `https://github.com/janikvonrotz/gocd-node-material.git`

A pipeline passes a material and its resulting artifacts through multiple stages. Each stage performs a build procedure or performs checsk on the artifact. Here is an example of a 4-stage pipeline:

1. build
2. test
3. analyze
4. publish

We are going to have configure a build and a test stage.

Name the current stage and job name *build*.

Select *More...* as task type and enter `yarn build` in the command field.

Finish the wizard and open the tab *stages*.

Click on *Add new stage* and enter *test* as stage and job name.

Then again select *More...* as task type and enter `yarn test` in the command field.

Save the new stage and unpause the pipeline.

![gocd pipeline in progress](/images/gocd pipeline in progress.PNG)

## Troubleshooting

**proxy settings**

In case the docker host runs behind a proxy and GoCD cannot connect to the git repository, make sure to configure the proxy environment variable when starting the docker container.

`docker run -e HTTPS_PROXY=https://_PROXY_HOST_:_PROXY_PORT_ -e HTTP_PROXY=http://_PROXY_HOST_:_PROXY_PORT_ ...`

**github connection refused**

If you cannot connect with the github repository and get this fatal error:

`fatal: unable to access 'https://github.com/janikvonrotz/gocd-node-material.git/': Failed to connect to github.com port 443: Connection refused`

It is most likely a proxy related problem.

**wrong version number**

Assuming the proxy env vars has been and you get this error:

`STDERR: fatal: unable to access 'https://github.com/janikvonrotz/gocd-node-material.git/': error:1400410B:SSL routines:CONNECT_CR_SRVR_HELLO:wrong version number`

It is most likely that git does not care about the env vars. Make sure to set the git proxy config for the go user:

```
su go
git config --global http.proxy https://_PROXY_HOST_:_PROXY_PORT_
git config --global https.proxy HTTP_PROXY=http://_PROXY_HOST_:_PROXY_PORT_
```

**gocd agent pending**

The GoCD agent entered the state pending even though there is no pipeline running.

Well, in my case the GoCD server could not connect to the agent, because a proxy was configured.

The issue was resolved by setting the git config proxy only for the GoCD server.