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

Using a GSM module and a thermo sensor the Raspberry we will retrieve temparature data and save it to a remote server.

I also gonna show how the data can be visualized and embedded on a web page.

# Hardware

The setup consists of the following components:

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

All this components should be available from your favorite electronics shop.

# Set up the Raspberry

The offical [Getting started with the Raspberry Pi](https://projects.raspberrypi.org/en/projects/raspberry-pi-getting-started) guide is everything you need to get the pi up and running.

For this tutorial I've used NOOBS to setup the os.

Once you've setup the pi, connect it to the internet and make sure everything is up-to-date.

# Assemble the thermo sensor

The layout is very simple:

![layout](/images/Project Lorauna/ds18b20_layout.png)



# Connect the pi

Enable the GSM module

# Read the temperature

Load the 1-Wire drivers by adding two modprobe commands to the modules config.

**/etc/modules**

```sh
...
modprobe w1-gpio
modprobe w1-therm
```

Look up the device.

```
cd /sys/bus/w1/devices/
ls
```

# Set up the mongoDB storage

**schema.js**

```js

```

**resolver.js**

```js

```

**curl.sh**

```sh

```

# Configure job to push data

Cron job that runs a python script

# Embed the thermo data

