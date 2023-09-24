---
title: "Running Discord Bots 24/7 for Free with Replit and Uptime Robot"
date: 2023-09-17
categories: Discord, Programming, Cloud
---

Have you ever wanted to keep your Discord bot running all the time without breaking the bank? In today's guide, we'll walk you through a straightforward process using Replit and Uptime Robot. These tools won't cost you a dime and come with the added bonus of a neat dashboard to monitor things.

## 0x01 - Setting Up Your Discord Bot

First things first, you need a basic Discord bot. If you already have one, great! If not, don't fret; there are many starter templates available online. Once you've got that, we'll make it ready for 24/7 uptime.

### 0x01A - Basic Discord  Bot Code

For newcomers, a basic bot code might look something like this:

```python
import discord
from discord.ext import commands

bot = commands.Bot(command_prefix='!')

@bot.event
async def on_ready():
    print(f'Logged in as {bot.user.name}')

@bot.command()
async def hello(ctx):
    await ctx.send('Hello there!')

 bot.run('YOUR_TOKEN')
```

Note: Replace `'YOUR_TOKEN'` with your bot token when you're ready to run. Which you can get by heading over to [Discord Developer Portal](https://discord.com/developers/applications) and creating a new application.

### 0x01B - **Introducing Keep-Alive**

Now, let's ensure the bot remains active. We'll use a small utility code, which essentially keeps the bot's server running. 

```python
from flask import Flask
from threading import Thread

app = Flask('')

@app.route('/')
def home():
    return "<b>Hack The Planet</b>"

def run():
  app.run(host='0.0.0.0',port=8080)

def keep_alive():
    t = Thread(target=run)
    t.start()
```

Save the above code as `keep_alive.py` in the same directory as your bot.

### 0x01C - Combining Keep-Alive With The Bot

Modify your bot's main code (likely named `main.py` or something similar) to use the `keep_alive` function. Here's how you can achieve this:

```python
import os
import discord
from discord.ext import commands
from keep_alive import keep_alive

bot = commands.Bot(command_prefix='!')

@bot.event
async def on_ready():
    print(f'Logged in as {bot.user.name}')

@bot.command()
async def hello(ctx):
    await ctx.send('Hello there!')

keep_alive()
bot.run(os.getenv("token"))
```

This setup ensures that the bot's server remains active, allowing Uptime Robot to regularly check its status.

## 0x02 - Hosting The Bot On Replit

[Replit](https://replit.com/) is an online IDE that supports various programming languages, including Python. For our purposes, it's a free platform where our bot can reside and run. 

[Insert Image of Replit Homepage]

- Sign up or log in to Replit.
- Click on the `+` sign to create a new Repl and select `Python`.
- Copy-paste your bot code and the `keep_alive` code into the respective files on Replit.
- Click on the `Run` button at the top. 

Now, your bot is running on Replit!

## 0x03 - Keeping Your Bot Awake With Uptime Robot

[Uptime Robot](https://uptimerobot.com/) is like a virtual guard. It will periodically check if your bot's server (set up by `keep_alive.py`) is active. If it's down, Uptime Robot will send a request to wake it up.

[Insert Image of Uptime Robot Dashboard]

Steps:

>- Head to Uptime Robot and create an account.
>- Click on `Add New Monitor`.
>- Choose `HTTP(s)` type.
>- In the URL box, add your Repl's URL (it should look like: `https://<YourReplName>.repl.co`).
>- Set the monitoring interval to 5 minutes.
>- Click `Create Monitor`, and you're set.

Uptime Robot will now ping your bot every 5 minutes, ensuring it remains awake.

## 0x04 - Wrapping Up

Congratulations! You've now set up your Discord bot to run round the clock without any maintenance. It's hosted for free, and you have a dashboard via Uptime Robot to monitor its uptime. 

This setup is ideal for those who wish for a hassle-free experience, without diving deep into the intricacies of hosting platforms and maintenance.

Remember to always respect Discord's Terms of Service when running bots, and happy coding!

