---
layout: post
title:  "How I made my home smart to save cost on electricty"
date:   2020-12-02 19:03:26 +0800
categories: home_assistant esp tasmota
---
Last summer we moved out from the city into the countryside of Hong Kong. Houses in HK are not quote following to highest building standards: No insulation, no central HVAC, single glazed windows, etc.
My daughter of 5 years was able to turn on the window unit AC in her room, but turning it off never crossed her mind. I was looking into ways to turn it off when there is no motion in her room for a while (and it's daytime). 

I was never impressed by people showing me their 'home automation' based on Alexa as I have privacy concerns with that. The following aspects were important to me:
* not cloud based; need to work without internet access
* open source; not dependent on some company that might turn of their servers at some point or start charging a fee.
* low price (we live here for rent for who knows how long and high CapEx cannot really be justified).

I ended up setting up a Home Assistant VM on my homelab (an already existing Thinkcentre tiny running Proxmox) and was hooked immediately. The Home Assistant community introduced me to:
* flashing alternative fimeware on so called 'Smart Plugs'
* flashing alternative fimeware on 4 USD bluetooth thermometers
* the universe of ESP8266 and ESP32 developer boards.

I started looking up smart plugs with the ES8266 chip as they can be flashed Over-the-Air. Unfortunately the vendors move away from these chipsets for exactly that reason, so make sure to do some research. You can look up devices on [templates.blakadder.com](https://templates.blakadder.com/2nice-UP111.html). I got myself 12 of the model  '2nice-UP111'.

They should be the same as [Gosund UP111-4-UK](https://www.amazon.co.uk/gp/product/B07ZSDWQQ8) but I cannot guarantee. I am waiting for a shipment of these and will report.

I flashed them using [tuya convert](https://github.com/ct-Open-Source/tuya-convert). I booted Kali Linux on my laptop as I something wouldn't want to work with on my Ubuntu system. There are different firmwares available. I used [Tasmota](https://tasmota.github.io/docs/) for my Smart Plugs.

Once they have been flashed they open up their own AP for configuration. Hook them up to your network and set up MQTT.
