---
title: " Nextcloud Social Login with Gitlab"
slug: nextcloud-social-login-with-gitlab
date: 2021-05-17T13:18:01+02:00
categories:
 - Identity and Access Management 
tags:
 - gitlab
 - nextcloud
 - openid connect
images:
 - "/images/building connection.jpg"
---

This example shows one way to configure [GitLab as an OpenID Connect (OIDC) identity provider](https://docs.gitlab.com/ee/integration/openid_connect_provider.html), so that only members a specific GitLab group are allowed to login.

<!--more-->

In the example, the GitLab group is named `hfmts` and the Nextcloud server is `https://nextcloud.example.com`.

## Setup GitLab

To make the login work with GitLab we need a GitLab application and a group. The GitLab application represents our Nextcloud application.

Open <https://gitlab.com/oauth/applications> and create a new application with these values:

Name: `Name of Nextcloud`\
Redirect URI: `https://cloud.example.com/apps/sociallogin/custom_oidc/gitlab`\
Condential: [ ]\
Scopes: `openid`

Save the *Application ID* and the *Secret* to a notepad. We need it later.

Next create the GitLab group at <https://gitlab.com/groups/new>. Enter a valid name and also not the generated url path. This will be the group name that is referenced in the Nextcloud integration.

## Setup Social Login

As we have our GitLab application and group, we are ready to setup the login integration in Nextcloud.

Open social login settings at <https://nextcloud.example.com/settings/admin/sociallogin>. Ensure the options are checked as follow:

* [ ] Disable auto create new users
* [ ] Create users with disabled account
* [ ] Allow users to connect social logins with their account
* [ ] Prevent creating an account if the email address exists in another account
* [x] Update user profile every login
* [ ] Do not prune not available user groups on login
* [ ] Automatically create groups if they do not exists
* [x] Restrict login for users without mapped groups
* [ ] Disable notify admins about new users

This will ensure that user data is always up-to-date and only members of the GitLab group are allowed to login.

Next create a new *Custom OpenID Connect* entry by clicking on the `+` next to it. Set these values

Internal name: `gitlab`\
Title: `GitLab`\
Authorize url: `https://gitlab.com/oauth/authorize`\
Token url: `https://gitlab.com/oauth/token`\
User info URL: `https://gitlab.com/oauth/userinfo`\
Client Id: `Copy from notepad`\
Client Secret:  `Copy from notepad`\
Scope: `openid`\
Groups claim: `groups`\
Button style: `Gitlab`\
Default group: `None`\
Group mapping: `hfmts` <--> `hfmts`

On the left of the *Group mapping* option is the displayname of the GitLab group and on the right is the selected Nextcloud group. Create a separate Nextcloud group if not already done.

Click *Save* on the bottom of the settings page.

## Test the login

To test the login integration proceed as followed:

* Log out of Nextcloud
* Click *Log in with GitLab*

Your are redirectred to GitLab.

* Sign into GitLab
* Authorize the application

You are redirected back to Nextcloud and are logged in.

Any group shares can be accessed immediately. This is a great way to work with external users which do not want have another account.