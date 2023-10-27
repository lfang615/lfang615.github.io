---
layout: post
title: "A Telegram app to trade crypto futures"
date: 2023-04-11
categories: kubernetes containers devops distributed kafka redis
thumbnail-img: ""
js:
  - https://cdn.jsdelivr.net/npm/mermaid@8.13.5/dist/mermaid.min.js
---

## Motivations

Like most all software projects that have come before, this one was also inspired by the need for further convenience. My adult attention span has been absorbed in trading on exchanges which offer crypto derivative products. Once you find yourself holding several accounts across different exchanges, you’ll quickly notice the waste involved in the repetitive actions taken to login to these various exchanges: fill out authentication credentials, verify my human form via captcha puzzle, and switching on your VPN provider (if you are trading on foreign exchanges).

The aim of this project was to eliminate that bit of waste (after some initial configurations which I’ll walk you through) by utilizing the API endpoints of my adopted choice of exchanges. Along with added convenience the experience also allows me to exercise with technologies involved in today’s microservice environments.


[Project Repo](https://github.com/lfang615/bybit-service)

## Description
 
This tool is comprised of 4 microservices, order manager, telegram bot, background service, Redis, and MongoDB. Their responsibilities are described below respectively, but in summary Telegram is used as the interface to submit crypto futures order to a separate service (order_manager),
built using the FastAPI framework which acts as a wrapper over the popular [CCXT](https://docs.ccxt.com/#/README) library.

1. order manager
  The popular ccxt library provides integrations to seemingly all crypto exchanges in operation currently and became an obvious choice since its needless to reinvent the wheel. I further abstracted the functionality provided by ccxt to limit interactions to only those exchanges which offer futures derivatives using a class named *Abstract Exchange*. You can add on future exchange integrations by deriving from this class. Further details are available in the README of the repository.
2. telegram bot
This service uses the telegram bot api to serve as the user interface to submit orders to order manager. Below shows examples of the available commands that can be used in the Telegram bot.
  - **/help**
    Displays a list of available commands.
