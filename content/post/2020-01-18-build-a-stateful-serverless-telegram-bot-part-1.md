---
title: "Build a stateful serverless Telegram bot - Part 1"
slug: build-a-stateful-serverless-telegram-bot-part-1
date: 2020-01-18T13:55:31+01:00
categories:
 - Software development
tags:
 - telegram
 - chatbot
 - serverless
 - stateful
 - redis
 - github action
 - now
images:
 - /images/chatbot.webp
---


For the first time in my life I bought a plant to decor my room. The plant is a [Monstera](https://en.wikipedia.org/wiki/Monstera), growing up to 2 meters if watered properly. It actually needs to be watered every two weeks. Now there are various options to remind myself of watering it. Installing an app, set an alarm on my phone or place a paper calendar next to the plant. But this is to easy, lets make it more complicated.
<!--more-->

More and more people start using Telegram. Telegram is a secure alternative to Whatsapp and most importantly it supports chatbots! For a while I've planed to build a bot. And by having the need to build a reminder I came up with the following idea: I am going to create a bot that manages the reminders for watering my plants. It will be hosted on the [Now](https://zeit.co) platform and I will use [GitHub Actions](https://github.com/features/actions) to schedule the reminders. The reminders are stored in a [redis database](https://redis.io/) and everything is MANAGED via the Telegram bot interface.

If you are interested on how I built this bot, keep on reading 😊.

If you wanna go straight for the code checkout [GitHub - HydroMeBot](https://github.com/janikvonrotz/hydrome-bot).

# Overview

In this tutorial I will proceed as followed:

Part 1:

* Register a bot
* Setup the now project
* Add a /start reply
* Deploy

Part 2:

* Setup redis connector
* Add /newreminder reply
* Add /listreminder reply
* Deploy

Assuming you want to build a similar bot, follow these instructions.

# Register a bot

On Telegram contact the `@BotFather`. This bot manages all bots on Telegram. Add a new bot with these details:

```
Name: ExampleBot
Description: Reminds you of watering your plants.
About: Reminds you of watering your plants.
Botpic: 🖼 has a botpic
Commands: 3 commands
```

Chose the bot username as you wish. In return you receive a Telegram bot token. We need it later on.

Add these commands to the bot:

```
start - Say hello
newreminder - Create a new reminder
listreminders - List all reminders
```

Note: In this tutorial I will not cover cases of mutating and deleting the reminders. They are implemented in the GitHub project. Once you finished part 2 it is easy to add new features to the bot.

# Setup the now project

Let's setup a new node project. We assume [now](https://zeit.co/download) is already installed and you are logged in with your ZEIT account.

Create a new project folder.

`mkdir example-bot && cd example-bot`

Create two subfolder.

`mkdir api db`

Note that now treats every file under `/api` as a serverless function.

Init npm and git.

`npm init && git init`

Create config files.

`touch .env .gitignore now.json`

Add packages with npm.

`npm install --save bluebird dotenv node-fetch redis`

Define the environment variables for the now deployment.

**now.json**

```json
{
  "env": {
    "TELEGRAM_TOKEN": "@hydrome_bot_telegram_token"
  }
}
```

Ignore the module folder and environment file for git.

`printf ".env\nnode_modules" >> .gitignore`

Update the environment file with the Telegram token.

**.env**

```bash
TELEGRAM_TOKEN=<your telegram bot token goes here>
```

Export the variable and create a now secret.

```bash
export $(cat .env | xargs)
now secrets add hydrome_bot_telegram_token $TELEGRAM_TOKEN
```

That's it for now 😉

## Optional tools

Optionally add a linter and test tool.

`npm install --save-dev ava standard`

I personally love [AVA](https://github.com/avajs/ava) and [StandardJS](https://standardjs.com/).

These packages require npm scripts.

**package.json**

```json
  ...
  "scripts": {
    "test": "ava",
    "lint": "standard"
  }
}
```

## Add a /start reply

We wanna teach our bot to reply to the opt-in command `/start`.

Add and index js file to the api folder with the following content.

**api/index.js**

```js
// Load env config
require('dotenv').config()

// Import modules
const start = require('./start')
const sendMessage = require('./send-message')

module.exports = async (req, res) => {
  console.log('BODY', req.body)

  // Check if Telegram message
  if (req.body && (req.body.message || req.body.callback_query)) {
    // Get message object
    var message = {}
    if (req.body.message) {
      message = req.body.message
    }
    // Callback query depends on request
    if (req.body.callback_query) {
      message = req.body.callback_query.message
      message.data = req.body.callback_query.data

      // Answer callback query
      sendMessage({
        callback_query_id: req.body.callback_query.id
      }, 'answerCallbackQuery')
    }

    // Build context
    const ctx = {
      request: null // await get(`request:${message.chat.id}`) -> See part 2
    }

    // Request is either current state if set or message text
    ctx.request = ctx.request || message.text

    // Match text request
    if (ctx.request.match('/start(.*)')) {
      await start(message, ctx)
    }
  }

  // Send default message
  res.end('This is the ExampleBot API.')
}
```

Every bot command has its own file. The API endpoint receives all bot messages and then forwards the message to appropriate command module.

Note: Callback queries are answered with an empty message. Callback queries are sent when the users presses a button on the inline keyboard in the bot chat. We will handle the dialog state differently therefore we do not need to answer callback queries.

Let's checkout the `sendMessage` method.

**api/send-message.js**

```js
const fetch = require('node-fetch')

// Sends messages to Telegram API
module.exports = async (body, method) => {
  const token = process.env.TELEGRAM_TOKEN

  // Get method optionally
  const useMethod = method || 'sendMessage'

  // Options for send message method
  if (useMethod === 'sendMessage') {
    // Disable notification
    body.disable_notification = !body.enable_notifications
  }

  return fetch(`https://api.telegram.org/bot${token}/${useMethod}`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(body)
  })
}
```

This module expects a telegram message body and the sends it to Telegram API. The API method can be overwritten by the optional parameter.

I assume this was easy to understand. The final file is the start module. As mentioned it simply replies to the `/start` command.

**start.js**

```js
const sendMessage = require('./send-message')

// Processes messages matching /start
module.exports = async (message) => {
  const chatId = message.chat.id

  await sendMessage({
    chat_id: chatId,
    text: '🙋 Hi, I am ExampleBot. Let me know if I should remind you of something.'
  })
}

```

# Deploy

Now we can deploy our bot with now. Run the `now --prod` in the root folder.

Once the deployment and building has finished you receive url like `https://example-bot.now.sh`.

As final step we need to set the webhook url. Once more export the Telegram token with `export $(cat .env | xargs)` and run this curl command: `curl https://api.telegram.org/bot$TELEGRAM_TOKEN/setWebhook?url=<your deployment url goes here>/api`

If the response was successful, open [Telegram](https://web.telegram.com) and start chatting with your bot.

Hope everything worked! See you in part 2 🏃
