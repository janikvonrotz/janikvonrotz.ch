---
title: SSH vs. VPN
slug: ssh-vs-vpn
date: 2024-11-11T11:45:58+01:00
categories:
  - Blog
tags:
  - vpn
  - ssh
  - security
  - comparison
images:
  - /images/logo.png
draft: true
---
Often when I deploy an application to a server owned by a customer or the customers IT provider, they require me to setup a virtual private network (VPN) connection. I then tell them about Secure Shell (SSH) protocol and how is better for this use case. As they are used to Windows server environment, where SSH is mostly absent, they insist on using VPNs.

In this post I will compare the two technologies and explain why SSH is the better option in this use case.

<!--more-->
### SSH

SSH is a protocol for secure connections.

With SSH you establish a connection from your computer to a remote server. 

![](../../../static/images/secure%20shell%20connection.png)

For user authentication SSH uses private/public keys. The user submits the public SSH key (think of a keylock) to the Operator of the SSH server. Only the user who owns the private key (also encrypted) can access the server.

The pros of SSH:

\+ Easy to setup on server
\+ Connects to one server only

The difficulties of SSH:

\- Requires named user on server
\- Different standards for private/public keys
\- No control of key management

### VPN

VPN is about a concept.

With a VPN you connect your computer to a remote network. The whole traffic of the computer is routed to the remote network.

![](../../../static/images/virtual%20private%20network.png)

There are various vendor for VPN software solutions and each of them is using different protocols and standards for 2 factor authentication (2FA).

* WireGuard
* IPSec
* OpenVPN
* SSTP

The pros of VPN:

\+ VPN solution is often bundled with Firewall
\+ UI clients for installation

The difficulties of VPN:

\- Exposes a network to the remote computer
\- Private traffic goes through corporate network
\- Different standards for 2FA

### Summary

In our use case we want to deploy a web application to a server. We only need access to a specific server and not other servers and services of the network. Therefore a SSH connection makes more sense.

A VPN makes sense when you want to access a corporate infrastructure in order to access your mail and file server. Or when you want to route your internet traffic from a different IP.