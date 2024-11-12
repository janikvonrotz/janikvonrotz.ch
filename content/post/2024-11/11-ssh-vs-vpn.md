---
title: SSH vs VPN
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
  - /images/ssh-vs-vpn.png
draft: false
---
When I deploy an application to a server that is owned by the customer or the customers IT provider, they very often require me to setup a virtual private network (VPN) connection. I tell them about Secure Shell (SSH) protocol and how is better fit for this use case. As they are used to Windows server environment, where SSH is mostly absent, they insist on using VPNs.

In this post I will compare the two technologies and explain why SSH is the better option in this use case.

<!--more-->
### SSH

SSH is a protocol for secure connections.

With SSH you establish a connection from your computer to a remote server. 

![](/images/secure%20shell%20connection.png)

For user authentication SSH uses private/public keys. The user submits the public SSH key (think of a keylock) to the Operator of the SSH server. Only the user who owns the private key (also encrypted) can access the server.

The pros of SSH:

\+ Easy to setup on server\
\+ Connects to one server only
\+ SSH is widely supported

The difficulties of SSH:

\- Requires user management on sever\
\- Different standards for private/public keys\
\- No control of key management

### VPN

VPN is a networking concept.

With a VPN you connect your computer to a remote network. The whole traffic of the computer is routed to the remote network.

![](/images/virtual%20private%20network.png)

There are various vendors for VPN software solutions and they are using different protocols and proprietary standards 2 factor authentication (2FA).

* WireGuard
* IPSec
* OpenVPN
* SSTP

The pros of VPN:

\+ Firewall bundled with VPN solution\
\+ UI clients for installation\
\+ Easy rollout for new users

The difficulties of VPN:

\- Exposes an entire network to remote computer\
\- Private traffic goes through corporate network\
\- Different standards for VPN and 2FA
### Summary

In our use case we want to deploy an application to a server. We only need access to one specific server of the remote network. In this case a direct connection with SSH make more sense.

A VPN is the better option when you require to access a corporate infrastructure and multiple server (mail, files, intranet, ...). Moreover, when you need privacy and need to hide your IP adress, a VPN can route your traffic to a different exit gateway.