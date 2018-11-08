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

We will start by buying the electronic components and finish with a chart showing the temperature data.

Walking through the tutorial requires basic knowledge in working with linux and advanced knowledge in building web applications. Whereas the web application part is optionally.

Using a GSM module and a thermo sensor our Raspberry Pi will retrieve temparature data and save it to a graphql server.

Using react and a graphql client library we will create a chart with the temperature data.

# Hardware

This is my hardware shopping list.

* Raspberry Pi 3 Model B
* MicrosSDHC Card 32 GB
* USB-C cable to power the Raspberry
* HDMI cable
* Mouse and keyboard
* Housing for the Raspberry
* Thermo sensor DS18B20+
* Adafruit breadboard PCB (optionally)
* 4.75k ohm resistor
* Jumper wire kit
* Altitude Tech IoT Bit Antenna
* Altitude Tech IoT Bit GSM HAT for the Raspberry Pi
* IoT SIM card

All this components should be available from your favorite electronics shop.

# Set up the Raspberry

The offical [Getting started with the Raspberry Pi](https://projects.raspberrypi.org/en/projects/raspberry-pi-getting-started) guide is everything you need to get the pi up and running.

For this tutorial I've used NOOBS to setup the os.

Once you've setup the pi, connect it to the internet via wifi and make sure everything is up and running.

# Assemble the thermo sensor

The thermo sensor must be connected with the gpio interface.

![layout](/images/Project Lorauna/ds18b20_layout.png)

I got help from a friend who helped me to solder the resistor and wires.

# Connect the pi

The GSM module is literally a hat for the Rasperry Pi. Stack it on top of the Pi, insert the SIM, plug in the usb cable and boot the Pi for installation.

```
installation instructions
```

# Read the temperature

Enable the gpio interface.

**/boot/config.txt**

```
...
dtoverlay=w1–gpio
```

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
cd 28-00000XXXXXXX
cat w1_slave
```

The temperature should be showed after `t=`.

# Set up the mongoDB storage



**schema.js**

```gql
type Temperature {
    _id: String
    created: Date
    value: Float
}

type Query {
    allTemperatures: [Temperature]
}
type Mutation {
    createTemperature(value: Float): Temperature
}
```

**resolver.js**

```js
...
Query: {
    allTemperatures: async (root, args, context) => {
        return (await context.db.collection('temperature').find({}).sort({created: -1}).toArray()).map(prepare)
    },
},
Mutation: {
    createTemperature: async (root, args, context) => {
        args.created = new Date()
        let res = await context.db.collection('temperature').insertOne(args)
        return prepare(res.ops[0])
    },
}
...
```

**/home/pi/sendTemeperatureData.sh**

```sh
# get temperature data from sensor
data=$(cat /sys/bus/w1/devices/28-00000a6a12a3/w1_slave | grep 't=' | cut -d "=" -f 2)

# convert into float
data=$(echo $data | sed 's/.\{2\}/&./')

curl \
  -X POST \
  -H "Content-Type: application/json" \
  --data '{ "query": "mutation { createTemperature(value: '$data') { _id, created } }" }' \
  https://example.com/graphql
```

# Configure job to push data

Login with the `pi` user and edit the crontab file.

`crontab -e`

Add the following entry to the crontab file, it will run the script every five minutes.

`*/5 *  * * *   /home/pi/sendTemperatureData.sh`

# Embed the thermo data

Show how the graphql server can be quried.