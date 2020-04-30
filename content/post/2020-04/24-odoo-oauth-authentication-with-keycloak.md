---
title: "Odoo OAuth authentication with Keycloak"
slug: odoo-oauth-authentication-with-keycloak
date: 2020-04-24T17:08:49+02:00
categories:
 - Identity and Access Management 
tags:
 - keycloak
 - odoo
 - oauth
 - authentication
images:
 - "/images/building connection.jpg"
---

# Introduction

OAuth is an authorization framework that enables applications to obtain limited access to user accounts on an HTTP service. It works by delegating user authentication to the service that hosts the user account, and authorizing third-party applications to access the user account.

In our scenario Keycloak acts as the OAuth service and Odoo as the application that delegates the user authentication. In this guide you learn how to configure Odoo and Keycloak to handle an implicit OAuth flow.
<!--more-->

![](/images/oauth%20implicit%20flow.png)

This image depicts what we want to achieve. The user accesses Odoo and then decides to authenticate with Keycloak. He gets forwarded to the login page and authorizes the Odoo application to access his account informations. He then gets redirected back to the application. Trust is enabled by only allowing selected applications to be redirected. If you want to know more about OAuth authentication head down to the source chapter.

We assume that we have the following service up and running:  
Keycloak Auth Server: `login.example.com`  
Odoo Application: `odoo.example.com`

Let's get started!

# Setup Keycloak client

Open the Keycloak management console, select your realm, navigate to *Configure > Clients* and create a new client.

For Client ID use `odoo`, for Client Protocol *openid-connect* and as Root URL enter `${authBaseUrl}`. Click save.

In the client edit view make the following configurations.

Access type: *confidential*

Odoo OAuth will pass a secret to intiate the login protocol.

Implicit Flow Enabled: *On*

Odoo OAuth requires the implicit flow.

Valid Redirect URIs:

* `/realms/example.com/account/*`
* `http://odoo.example.com/auth_oauth/signin`
* `http://localhost:8069/auth_oauth/signin` *for development purposes*

Allow redirection to the odoo login page.

Base URL: `/realms/example.com/account/`

Leave the *Admin URL* and *Web Origins* empty.

Save the settings and open the *Mappers* tab. Click on *Add Builtin*. Select and add the `email` entry. Open the *email* mapper and set as *Token Claim Name* the value `user_id`.

This will ensure that the token has the email address set as user id.

# Update OAuth module

In order to authenticate with Keycloak the Odoo OAuth module requires some changes. I have created a Odoo module that applies the necessary changes.

Download the module zip from [https://github.com/Mint-System/Odoo-App-Auth-OAuth-Keycloak](https://github.com/Mint-System/Odoo-App-Auth-OAuth-Keycloak) and install the module.

This module makes the following changes:

* Add new field `x_keycloak` to the `auth.oauth.provider` model.
* Update the `auth_oauth.view_oauth_provider_form` view  with the field *Keycloak*.
* Override the `_auth_oauth_rpc` and `_auth_oauth_validate` methods of the `res.users` class.

The new methods support the bearer access token format and therefore make the authentication with Keycloak possible.

# Add Keycloak provider in Odoo

Open the Odoo dashboard and navigate to *Settings > General Settings > Integrations*. If necessary enable *OAuth Authentication* and then click on *OAuth Providers*. Create a new provider with the following settings:

Provider Name: `Login Example`  
Client ID: `odoo`  
Allowed: [ x ]  
Keycloak: [ x ]  
Body: `Login Example`  
Authentication URL: `https://login.example.com/auth/realms/example.com/protocol/openid-connect/auth`  
Validation URL: `https://login.example.com/auth/realms/example.com/protocol/openid-connect/userinfo`  

# Update portal user template

By default Odoo creates a portal user for users that sign in via OAuth. We want Odoo to create a internal user instead.

Enable the developer mode and navigate to *Settings > General Settings > Users > Customer Account > Default Access Rights*. Edit the template and select for *User types*: *Internal User*. Save the portal user template.

# Test the login

In Keycloak ensure that a user has been added to the realm. If necessary create one with password credentials.

Logout out of Odoo. You should now see a button *Login Example*. Click on it and wait to be forwarded. Log in an await to be redirected. If everything has been configured properly you should now be logged in and see the Odoo activity page.

# Source

[Odoo Help - Keycloak Authentication on v12](https://www.odoo.com/de_DE/forum/hilfe-1/question/keycloak-authentication-on-v12-146506)  
[Digital OCean Tutorials - An Introduction to OAuth 2](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2)