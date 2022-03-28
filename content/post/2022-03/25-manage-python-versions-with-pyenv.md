---
title: "Manage Python versions with pyenv"
slug: manage-python-versions-with-pyenv
date: 2022-03-25T16:00:45+01:00
categories:
 - Python
tags:
 - python
 - pyenv
 - version management
images:
 - /wp-content/uploads/2015/10/Python-Logo.png
---

With [pyenv](https://github.com/pyenv/pyenv) you can easily switch between python versions. When working on multiple project is recommended to create a `.python-version` file containing the targeted node version. Here is how.

<!--more-->

First make sure the right python version is installed.

```bash
➜  example-project git:(main) python --version
Python 3.7.8
➜  example-project git:(main) pyenv install 3.8.6
Installing Python-3.8.6...
Installed Python-3.8.6 to /home/janikvonrotz/.pyenv/versions/3.8.6  
```

Use the new version locally.

```bash
➜  example-project git:(main) pyenv local 3.8.6
➜  example-project git:(main) ✗ cat .python-version
3.8.6
```

Of course you can set the new version globally using `pyenv global x.x.x`.

Switch between folders to check if the correct version is loaded automatically.

```bash
➜  ~ python --version 
Python 3.7.8
➜  example-project git:(main) ✗ python --version
Python 3.8.6
```

Here is an overview of the python version is selected when working with pyenv.

![pyenv-pyramid](/images/pyenv-pyramid.webp)