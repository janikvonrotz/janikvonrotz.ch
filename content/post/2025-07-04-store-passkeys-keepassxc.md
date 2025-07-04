---
title: Store Passkeys in KeePassXC
slug: store-passkeys-keepassxc
date: 2025-07-04T08:04:42+02:00
categories:
  - Security
tags:
  - keepass
  - passkeys
  - password
  - manage
images:
  - /images/KeePassXC.png
draft: false
---
---
The goal of Passkeys is to replace passwords.

The idea is that instead of remembering a password and entering it to access your account, you own a device that generates a password for you.

Remembering is replaced with Owning.

In this post, I'll give an example of such a device and how you can create and store a Passkey securely.

<!--more-->

## YubiKey

A device that can be used as a Passkey is the [YubiKey](https://en.wikipedia.org/wiki/YubiKey). Setting it up is actually quite simple. In this example, we have an online account and are going to set up the Passkey. In the security settings of the online account, I click *Add Passkey*, and the browser prompts for an input:

![](/images/browser%20prompt%20passkey.png)

The YubiKey is plugged in, I touch the YubiKey, and the device is registered. That's it. From now on, when I log into my account, as a login option, I can choose Passkey. The browser prompts for the input, I touch the YubiKey, and get logged in.

However, at this point, you might ask: What happens when I lose the YubiKey? Can I make a backup of the key?

The short answer is: You cannot create a backup of a YubiKey.

So, the YubiKey might not be the best solution to use as a Passkey. Luckily, there are other providers and devices to manage a Passkey.

## KeePassXC

Here, I will show you how you can create and store a Passkey with [KeePassXC](https://keepassxc.org/).

* Install KeePassXC
* Set up a password database
* In the settings, enable the browser integration and select your browser
* Install the KeePassXC browser extension
* Connect the extension to your KeePassXC database
* Enable the *Passkeys* option in the KeePassXC browser extension settings
* Optionally, set the *Default group for saving new passkeys* field

![](/images/KeePassXC%20Browser%20Extension%20Passkeys.png)

You may need to restart your browser for the changes to take effect. When you register a new Passkey for your account, the KeePassXC extension will open the KeePassXC database locally and show this dialog:

![](/images/KeePassXC%20Passkey%20Register.png)

You can click *Register*, and then you'll find a Passkey entry in your KeePassXC database.

Logging into your account with a Passkey is simple. Select the Passkey option, and the KeePassXC extension will find a matching entry and prompt to authenticate.

![](/images/KeePassXC%20Authentication.png)

Click *Authenticate*, and you should be logged in.

The KeePassXC database can be backed up, and while Passkey entries can technically be exported, it's crucial to understand the security implications before doing so.

## Recommendations

When registering a Passkey for your account, I recommend using both KeePassXC and YubiKey. The main reason is the mobile browser of your smartphone. Setting up the KeePassXC Passkey solution on your smartphone is currently not possible (as far as I know).

The YubiKey can be plugged into the smartphone's USB-C port and used to authenticate with any mobile browser. Here is a simple webpage that shows Passkey device and browser compatibility: <https://www.passkeys.io/compatible-devices>
