---
title: Node.js dynamic settings
date: 2016-08-25T19:43:31+00:00
author: Janik von Rotz
permalink: /2016/08/25/node-js-dynamic-settings/
dsq_thread_id:
  - "5094803262"
image: /wp-content/uploads/2014/03/Node.js-Logo.png
categories:
  - Node.js
tags:
  - beginner
  - configuration
  - environment
  - files
  - load
  - node
  - nodejs
  - settings
  - variables
---
As I'm relatively new to Node I had to wrap my head around a very basic thing. Getting external variables into my app. Loading settings for different environments from config files and via environment variables (for heroku deployment) should supposedly be an easy challenge. However, none of the solutions I've found were well enough for my scenario:

* Store config in json file.
* Have different configuration files (development, production, ...).
* Load configs from environment variables.
* Fallback from environment to config file.
* App lives in one folder.
<!--more-->

By scrapping together the best of different approaches I've found the perfect solution for my case. All the files showed below are stored in the same directory, which might be the root directory.

**app.js**

```
...
var config = require('./config');
...
```

The file where your app lives. Simply call the config.js file which gets the configs for your app.

**config.js**

```
var fs = require('fs');
var config = {}
process.env.NODE_ENV = process.env.NODE_ENV || 'development';
var path = `./config-${process.env.NODE_ENV}.json`
if (fs.existsSync(path)) {
  config = require(path);
}
module.exports = config;
```

Based on the NODE_ENV variable this code loads json data and exports them. If the settings file doesn't exist, simply return an empty object.

**config-development.json**

```
{
  "LUIS_APP_URL": "https://api.projectoxford.ai/luis/v1/application?id=324234234233424242342",
  "PORT": 443,
  "MICROSOFT_APP_ID": "xxxxx",
  "MICROSOFT_APP_PASSWORD": "xxxxx"
}
```

An example for config file. Make a copy with different settings for each environment.

**main.js**

```
...
server.listen(process.env.PORT || config.PORT || 3978, function () {
   console.log('%s listening to %s', server.name, server.url);
})
...
```

A final example of how a configuration variable might be accessed within your app.