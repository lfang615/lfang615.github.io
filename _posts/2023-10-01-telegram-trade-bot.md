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
 
This tool is comprised of 5 microservices, order manager, telegram bot, background service, Redis, and MongoDB. Their responsibilities are described below respectively, but in in a nutshell Telegram is used as the user interface to submit crypto futures orders to a separate service (order manager), built using the FastAPI framework which acts as a wrapper over the popular [CCXT](https://docs.ccxt.com/#/README) library, exposing actions that can be taken on your chosen exchanges.

1. **Order Manager**
  - Exposes the functionality that can be taken on configured exchanges (built using FastAPI). It utilizes the popular ccxt library, which provides integrations to seemingly all crypto exchanges in operation currently and became an obvious     choice since its needless to reinvent the wheel. I further abstracted the functionality provided by ccxt to limit interactions to only those exchanges which offer futures derivatives using a class named *Abstract Exchange*. You can add     on future exchange integrations by deriving from this class. Further details are available in the README of the repository.

2. **Telegram Bot**
  - This service uses the telegram bot api to serve as the user interface to submit orders to order manager. Below shows examples of the available commands that can be used in the Telegram bot.
    
    - **/help**: Displays a list of available commands.
      - ![](/assets/img/help_command.png)
        
    - **/start**: Displays the list of order types that can be executed. After selecting an order type, a valid JSON template will be returned which needs to be copied, pasted, and edited to your needs inside the chat window.
      - ![](/assets/img/start_command.png)
    
    - **/submit**: Submits order to exchange using the JSON template provided by your selection choice from the */start* command, described above.
      - ![](/assets/img/submit_command.png)

3. **Background service**
  - Calls the external endpoints of the configured exchanges to retrieve tradeable symbols and associated details regarding the minimum and maximum leverage that can be used, repeated on a 24 hour interval.


4. **MongoDB**
  - Used to store user credentials needed to authenticate, api keys and secrets for chosen exchanges, and order submission details.


5. **Redis**
  - Serves as a cache to store the authentication token needed for order manager’s (Fast API) authenticated endpoints after authentication credentials and various symbol details for integrated exchanges.

6. **Kafka**
  - This topic queue is included in the docker-compose file but it currently isn't being utilized. Perhaps I'll use it to store all my opening and closing orders and have some tool (unknown for now) retrieve the data from there so I can further analyze my P&L performance across exchanges.

## Instructions to run locally

**Repository Links:**
- [order manager](https://github.com/lfang615/crypto_trading_service)
- [telegram bot](https://github.com/lfang615/telegram_crypto_futures) 

**Configuring order manager**

After cloning the repository, navigate to the directory and execute `docker-compose up -d` inside a terminal shell to start the services.

There are python scripts located under */dev_utilities* for creating a user and adding your chosen exchanges api keys and secrets.

1. Create a user
Inside the project directory execute the following:
``` shell
python3 seed_user.py --username <username> --password <password>
```

2. Integrating exchanges
As mentioned earlier under [Motivations](#Motivations), this app aims to limit the integrations provided by [ccxt](https://docs.ccxt.com/#/README) to only those which offer futures contracts. The integrations I have included so far are limited to Bybit, Bitget and MEXC. To integrate other exchanges, create a class deriving from **AbstractExchange** located in */app/exchanges/integrations.py*

To integrate an exchange, browse to the exchanges platform and go through the steps to generate an api key and secret for your account. Add the key and secret to the application by executing the following command that is again, located under */dev_utilities*.
```shell
python3 update_user.py <name of exchange> --apikey <apikey> --secret <api_secret> --username <username>
```
    
    
