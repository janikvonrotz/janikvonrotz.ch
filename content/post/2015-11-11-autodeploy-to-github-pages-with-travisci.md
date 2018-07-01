---
id: 3631
title: Autodeploy to Github Pages with Travis CI
date: 2015-11-11T09:37:05+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=3631
permalink: /2015/11/11/autodeploy-to-github-pages-with-travisci/
dsq_thread_id:
  - "4308594906"
image: /wp-content/uploads/2015/11/Travis-CI-logo.jpg
categories:
  - Travis
tags:
  - continuous
  - github
  - integration
  - pages
  - testing
  - travis
---
[Github pages](https://pages.github.com/) are a common way to create a website for your open-source project.
Github provides a website generator tool which simply reads the README file from your repository.
The big disadvantage is that there's no auto deployment for your this kind of website. You always have to run the generator manually.
To solve this problem we need a third-party computing engine that automatically updates our page. This is the perfect job for [Travis CI](https://travis-ci.org).
Travis CI is a Github integrated continuous testing platform that runs tests for different development frameworks. It triggers a test whenever you make a commit on your Travis registered repository. We will use Travis to update our page whenever we make a commit to our repository.
<!--more-->

Now I will show you how you can setup this kind of autodeploy. This guide assumes you have a repository on Github that uses npm and gulp to create a website into a folder named `out`. Make sure to update <> tags in the instruction steps.

* First create a `deploy.sh` file in your repository.

This script will be executed by Travis CI later. The idea of the script is to create an `out` folder which holds the website that is deployed to the Github page repository.

```bash
#!/bin/bash
set -e # exit with nonzero exit code if anything fails

# clear and re-create the out directory
rm -rf out || exit 0;
mkdir out;

# run commands to build the website in the out folder
npm install -g gulp
npm install
gulp

# go to the out directory and create a *new* Git repo
cd out
git init

# inside this git repo we'll pretend to be a new user
git config user.name "Travis CI"
git config user.email "<name>@<domain>.com"

# The first and only commit to this new Git repo contains all the
# files present with the commit message "Deploy to GitHub Pages".
git add .
git commit -m "Deploy to GitHub Pages"

# Force push from the current repo's master branch to the remote
# repo's gh-pages branch. (All previous history on the gh-pages branch
# will be lost, since we are overwriting it.) We redirect any output to
# /dev/null to hide any sensitive credential data that might otherwise be exposed.
git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" master:gh-pages > /dev/null 2>&amp;1
```

As already mention this script uses npm to installs gulp and dependencies and also runs the gulp tasks that will in result create the website.

* Next register your repository on: [https://travis-ci.org/](https://travis-ci.org/)
* Then go to [https://github.com/settings/tokens](https://github.com/settings/tokens) and create a access token with default rights for Travis CI.
* Now format the token like this: `GH_TOKEN=<your_github_username>:<your_token>`
Travis will use this token to update the git repository. To provide the token to Travis we can set environment variables. This environment variables are declared in the `.travis.yml` file.

* Install the Travis cli.
You can either install it with npm: [https://www.npmjs.com/package/travis-encrypt](https://www.npmjs.com/package/travis-encrypt)
Or as ruby gem: [https://github.com/travis-ci/travis.rb](https://github.com/travis-ci/travis.rb)
Either way the Travis cli can encrypt strings by using the Travis ssl certificate of your repository.

* Open a command line in your repository (you have to be in your repository otherwise Travis cannot encrypt anything).
* Run `travis encrypt GH_TOKEN=<your_github_username>:<your_token>` and copy the encrypted string.
* Create a `.travis.yml` in your repository and paste the code below with your encrypted string.

```
script: bash ./deploy.sh
env:
  global:
  - GH_REF: github.com/<username>/<repository>.git
  - secure: "<encrypted_token>"
```

This configuration file will tell Travis what to do and set environment variables which will passed to our `deploy.sh` script file.

* Make a commit on your repository, go to [https://travis-ci.org/username/repository](https://travis-ci.org/username/repository) and make sure the build is successful.

If you want to see an example in action open this repository: [https://github.com/HaloCustomEdition/Halo](https://github.com/HaloCustomEdition/Halo)
