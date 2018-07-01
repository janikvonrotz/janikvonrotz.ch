---
id: 3953
title: How to publish a react component to the npm directory
date: 2016-06-09T12:33:36+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=3953
permalink: /2016/06/09/how-to-publish-a-react-component-to-the-npm-directory/
dsq_thread_id:
  - "4896495057"
image: /wp-content/uploads/2014/04/React-Logo.png
categories:
  - Blog
  - Node.js
  - React
tags:
  - babel
  - best
  - compile
  - component
  - index
  - javascript
  - node
  - npm
  - packagage
  - practice
  - publish
  - source
  - transpile
---
This time I'll show you how to publish react components to the npm directory. This guide does not cover testing it only shows the easiest way to use your components for different projects as npm dependency.

<!--more-->

Open your project folder with a CLI.

Initialize a new npm project.

    npm init

Answer the prompts and finish the initialization. Make sure the name of the project is unique in the npm directory.

Next install react.

    npm install react --save

We will use ES6 und JSX to write the react components.

These standards are not supported by default. We will use babel and its presets to transpile ES6 and JSX code into pure vanilla JavaScript.

    npm install babel-cli babel-preset-es2015 babel-preset-react --save-dev

Next we tell babel how to convert the code. For this create a `.babelrc` file in the root with this content:

```
{
  "presets": ["es2015", "react"]
}
```

To publish and transpile our react component code add two scripts to the `package.json` file.

```
"scripts": {
  "prepublish": "npm run build",
  "build": "babel ./src --out-dir ./dist"
},
```

If you run `npm build` babel will compile everything from `/src` to `/dist`.

Create the `/src` folder and add a react component like `HelloWorld.jsx`:

```
import React from 'react';

export default class JelloWorld extends React.Component {
  render() {
    return (
      <div>Hello world!!</div>
    )
  }
}
```

Before publishing this hello world component run `npm run build` just to see how babel transpiles the scripts.

To let npm know which files are part of the npm package create a index file `/src/index.jsx`.

```
import HelloWorld from './HelloWorld';

export { HelloWorld };
```

Make sure to update the main attribute in the `package.json` file.

```
"main": "./dist/index.js",
```

When running `npm publish` it will automatically transpile the scripts and publish it to the npm repository.

But before doing so you have to login with an npm Account.

    npm login
    npm publish

If everything went well you can start adding this component as dependency in your app.

    npm install your-pojrect-name --save

In your react app import the component as showed below.

    import { HelloWorld } from 'your-project-name';

When updating you project and republish it again make sure to increment the semantic version of the project.

    npm version patch

This will increment the version from `1.0.0` to `1.0.1`.

In case you do not want to publish the JSX source add and `.npmignore` file.

```
/src
.babelrc
```

In case you are using git to host your project add a `.gitignore` file.

```
/node_modules
```

# Source

[http://rajasekarm.com/publishing-react-component-npm/](http://rajasekarm.com/publishing-react-component-npm/)

[http://jamesknelson.com/the-six-things-you-need-to-know-about-babel-6/](http://jamesknelson.com/the-six-things-you-need-to-know-about-babel-6/)

[http://tstringer.github.io/npm/npmjs/2015/10/29/npm-incrementing-version.html](http://tstringer.github.io/npm/npmjs/2015/10/29/npm-incrementing-version.html)
