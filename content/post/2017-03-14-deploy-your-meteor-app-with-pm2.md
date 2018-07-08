---
title: Deploy your Meteor app with PM2
date: 2017-03-14T14:12:42+00:00
author: Janik von Rotz
permalink: /2017/03/14/deploy-your-meteor-app-with-pm2/
dsq_thread_id:
  - "5631475555"
image: /wp-content/uploads/2017/02/Meteor-Logo-inverted.png
categories:
  - Meteor
tags:
  - configuration
  - deploy
  - management
  - meteor
  - pm2
  - process
---
PM2 is a well-known node process manager. Not so well-known is its deployment feature. With pm2 you can reasonable easy deploy any node application. As Meteor is always a node app at its heart I've decided to deploy my current Meteor project with PM2.
<!--more-->
I assume you already have PM2 installed on your client and server, otherwise checkout [http://pm2.keymetrics.io/](http://pm2.keymetrics.io/).

To create a deployment configuration file run `pm2 ecosystem` in your project folder. This will create the `ecosystem.config.js` file. You can now configure this file according to your projects environment.

Here's mine:

**ecosystem.config.js**

```
module.exports = {
  /**
   * Application configuration section
   * http://pm2.keymetrics.io/docs/usage/application-declaration/
   */
  apps: [
    {
      name: 'Project',
      script: './bundle/main.js',
      env: {
        NODE_ENV: 'dev',
        MONGO_URL: 'mongodb://user:password@id.mlab.com:port/database',
        ROOT_URL: 'https://hostname.example.com',
        PORT: 3000,
        METEOR_SETTINGS: {
          'private': { ... },
          'public': { ... }
        }
      },
      env_prod: {
      },
    },
  ],

  /**
   * Deployment section
   * http://pm2.keymetrics.io/docs/usage/deployment/
   */
  deploy: {
    dev: {
      user: 'login',
      host: 'hostname.example.com',
      key: '/path/to/your/ssh/id_rsa',
      ref: 'origin/develop',
      repo: 'git@gitlab.com:username/repo.git',
      path: '/path/to/your/project/folder',
      'post-deploy': 'rm -rf ./bundle && npm install --production && meteor build . --directory && cd ./bundle/programs/server && npm install --production && cd ../../.. && pm2 startOrRestart ecosystem.config.js',
    }
  }
}
```

**Hint**: If your git repo is private, make sure your store a ssh key with access rights to the repo on your server.

The `post-deploy` command will build the node app from your Meteor project. This command once executed takes some time to finish.

When you're finished updating the config file, you have to set up the project folder on your server.
Run `pm2 deploy ecosystem.config.js dev setup` to do so. Your repo will be cloned and launched on your server.

From now on, whenever you've pushed new changes to your repo, you can deploy your app with `pm2 deploy ecosystem.config.js`.

![Untitled](/wp-content/uploads/2017/03/Screen-Shot-2017-03-14-at-13.44.04.png)...
{{< figure src="/wp-content/uploads/2017/03/Screen-Shot-2017-03-14-at-13.44.30.png" title="Meteor app deployed successfully." >}}

Make sure to have a look on the [PM2 deployment documentation](http://pm2.keymetrics.io/docs/usage/deployment/). 


