---
title: "2018 09 30 Raspberry Pi thermometer that connects from anywhere via GSM"
slug: 2018-09-30-raspberry-pi-thermometer-that-connects-anyywhere-via-gsm
date: 2018-09-24T20:00:23+02:00
categories:
 - Blog
tags:
 - hello
 - world
image: /images/logo.png
draft: true
---

In this tutorial I am going to show how you can build an online accessible thermometer using the Raspberry Pi 3 B-model.

Using a GSM module and a thermo sensor the Raspberry will send temparature data to a server periodically.

The data is stored within a database.

Finally, I will show how the data can be visualized and embedded on a web page.

# Hardware

The whole setup requires quite a few components:

* Raspberry Pi 3 Model B
* MicrosSDHC Card 32 GB
* USB-C cable to power the Raspberry
* HDMI cable
* Mouse and keyboard
* Housing for the Raspberry
* Thermo sensor DS18B20+
* Adafruit breadboard PCB
* 4.75k ohm resistor
* Jumper wire kit
* Altitude Tech IoT Bit Antenna
* Altitude Tech IoT Bit GSM HAT for the Raspberry Pi
* Prepaid SIM card

All this components should be available at your favorite electronics shop.

# Set up the Raspberry

The offical [Getting started with the Raspberry Pi](https://projects.raspberrypi.org/en/projects/raspberry-pi-getting-started) guide is everything you need to get the pi up and running.

For this tutorial I've used NOOBS to setup the os.

Once you've setup the pi, connect it to the internet and make sure everything is up-to-date.

# Assemble the thermo sensor

In this part we are going to assemble the themo sensor, connect it to the pi and read the temparature data.

![layout]()

![breadboard pins]()

# Connect the pi

Enable the GSM module

# Read the temperature

# Set up the mongoDB storage

https://docs.mlab.com/data-api/

# Configure job to push data

Cron job that runs a script

# Embed the thermo data