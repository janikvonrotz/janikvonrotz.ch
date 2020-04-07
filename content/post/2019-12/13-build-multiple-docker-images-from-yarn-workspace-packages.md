---
title: "Build multiple Docker image from Yarn workspace packages"
slug: build-multiple-docker-images-from-yarn-workspace-packages
date: 2019-12-13T18:17:28+01:00
categories:
  - Continuous delivery
tags:
 - docker
 - yarn
 - monorepo
images:
 - /images/shipping container.jpg
---

If you decide to manage your node application repository as a monorepo you sure heard about [Yarn workspaces](https://yarnpkg.com/lang/en/docs/workspaces/). Yarn has an out-of-the-box support for managing multiple packages in a single codebase. Using yarn workspaces package dependencies can be centralized and packages can reference each other. Tasks can be executed for all packages at once. It solves various build related problems for a monorepo.
<!--more-->

However, if you want to build docker images for one package only things start to become more complicated. Let's assume we have a workspace with the following configuration:

**package.json**

```json
{
  "name": "example",
  "version": "1.0.0",
  "scripts": {
    ...
  },
  "private": true,
  "workspaces": ["api", "app", "mongo"],
  "author": "Janik von Rotz",
  "license": "MIT"
}
```

There are three packages: api, app and mongo.

The api and app packages can be run independently.

However, the api package depends on the mongo package.

**api/package.json**

```json
{
  "name": "api",
  "version": "1.0.0",
  "scripts": {
    ...
  },
  "dependencies": {
    ...
    "mongo": "1.0.0"
  },
...
```

As you can see package dependencies do not differ from npm dependencies.

Assuming we would like to build a docker image for the api package, we need to ensure that the app code does not end up on the image. Usually one would simply `cd api;docker build .`, but here we need extra preparation to resolve the `"mongo": "1.0.0"` package dependency.

The solution I came up with is not extraordinary. I decided to build the image from the root folder. First I copy the workspace config. Then copy the required packages and build them using the `yarn workspace` command. Finally I switch the work dir into the package folder and start the server.

This is the Docker file for the api package:

**api/Dockerfile**

```
FROM node:10-alpine

# Create app directory
WORKDIR /usr/src/

# Copy workspace config
COPY ./package*.json .

# Copy packages
COPY ./api ./api
COPY ./mongo ./mongo

# Install dependencies for packages
RUN yarn workspace mongo install
RUN yarn workspace api install

# Run the app
WORKDIR /usr/src/api
EXPOSE 3000
CMD ["yarn", "start"]
```

The image is built by running the command `docker build -t api:1.0.0 -f api/Dockerfile .` from the root folder.

In similar fashion docker files can be build for the other packages.
