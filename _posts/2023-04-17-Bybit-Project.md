---
layout: post
title: "Tool for trading cyprto derivatives on multiple platforms"
date: 2023-04-11
categories: kubernetes containers devops distributed kafka redis
thumbnail-img: ""
js:
  - https://cdn.jsdelivr.net/npm/mermaid@8.13.5/dist/mermaid.min.js
---

## Motivations

When taking upon an interest in capital markets, you'll sooner or later bear the burden of navigating the regulations that govern these markets. Then if your curiousities ever lead you to browse around and compare financial services offered in comporable developed first world nations, it'll become apparent (*not much analytical skills needed*) that the U.S. is much more strict as to how they allow their residents to partipate in the markets.

For those among the crypto crowd in the U.S. researching crypto exchanges offering derivitive products, you may have narrowed in on Bybit as a suitable candidate for your trading activities, since their KYC policies are more liberal compared to the other players in the space. They provide 2 separate platforms, one for U.S. residents and another for everyone else abroad. For those with a little technical know how, I'm sure you can figure out how to access the latter.

Using the method above pigeon holes you to their web app to make trades which is only made available in the desktop version of their platform, making usability a nightmare. Mobile VPN connections, in my experience can't be relied on to provide stable connections (*from my provider at least*) for long durations which causes the site the session to become invalid and ultimately creates an annoying loop of reconnecting to VPN server, inputing login details, fetching the multifactor code from Google Authenticator, every time I want to place a trade when I'm away from the a desktop. This project was started to resolve those annoyances and to explore various tools used in distributed systems.

*Update 4-30-2023*
Just a few days after beginning the project, Bybit announced that they will be adopting KYC measures for all current and future users of the platform, thereby negating my original motivation for developing this tool in the first place. However it did force me to think of ways to repurpose the functionality that had already been developed. Instead of limiting the tool to interface with one exchange I applied one of the guiding principles of clean software development, extensibility and created an interface defining the common ordering, market data fetching, and account balance capabilities that are common to all trading platforms, irrespective of any particular asset.

*Update 05-15-2023*
After laboring through the rewrite and associated unit tests to use the interface mentioned in the previous update I found the [CCXT](https://docs.ccxt.com/#/README) package which is a popular and well supported package which provides a common interface to interact with all currently relevant crypto exchanges (everything I required and more, developed better). That discovery served as a frustrating but necessary reminder to always research to see what's available before you needlessly reinvent the wheel. *Currently updating the project repo to use the CXXT library*.


[Project Repo](https://github.com/lfang615/bybit-service)

## Description
 
- ~~Motivations were described above, but the primary utility was to have a self hosted layer I can use to interact with Bybit.~~
- Application that allows you to execute, view and manage orders with any exchange supported by [CCXT](https://docs.ccxt.com/#/README) which offers derivitive/contract trading.
- The interface in the project is utilized as a wrapper for the common [CCXT](https://docs.ccxt.com/#/README) functions associated with dervitives orders.
  - Currently only concrete implementations for Bitget and Bybit are available.
- Microservice architecture, responsibilities of each service are described in further detail in the section below.
- *Update 05-20-2023* - Has been updated with an abstract class which utilizes the [CCXT](https://docs.ccxt.com/#/README) package. CCXT provides a common structure that can be used to interact with an impresive list of crypto exchanges (*wish I found ccxt when I started the project*).
- The README in the project repo will contain a list of the available features.

### Communication Diagram

<div class="mermaid">
sequenceDiagram
    participant order_manager
    participant kafka_topics
    participant redis
    participant order_resolver
    participant bybit
    activate order_manager
    Note left of order_manager: order submitted
    order_manager-->>bybit: /contract/v3/private/order/create {orderLinkId}
    bybit-->>order_manager: Success/Fail
    alt Fail
        order_manager->>order_manager: Display Error
    else Success
        order_manager-->>kafka_topics: orders_executed
    end
    loop consume from orders_executed
        kafka_topics-->>order_resolver: "Set orderLinkId in orders cache"      
    end
    loop Check for order status changes
        order_resolver-->>bybit: Status changes for orderLinkId?
    end
    bybit-->>order_resolver: Order Response
    loop Update orders and/or positions caches
        alt Status in ["Filled", "Cancelled", "Deactivated", "Triggered"]
            order_resolver-->>redis: Update orders:orderLinkId and BREAK BGTask
        else Status in ["Filled", "PartiallyFilled"]
            order_resolver-->>redis: Update positions:symbol:side
        else order[orderStatus] != orderStatus
            order_resolver-->>redis: Update orders:orderLinkId
        end
    end
    order_resolver-->>kafka_topics: Send updated order to orders_resolved
    order_resolver-->>kafka_topics: Send updated position to positions_updated
</div>

### Overview of each service and their responsibility

1. **order_manager**
  - Accept and validate order details. Routes order to appropriate exchange
  - Contains Kafka producer which sends successfully submitted order to *orders_executed* topic. 
  - Two WebSocket available for connection
    1. */ws/orders* - Updates frontend with updated orders consumed from *updated_orders* Kafka topic. In the event of the WebSocket disconnecting, data will be streamed from an offset defined at the moment of disconnection.
    2. *ws/positions* - Updates frontend with updated position information. Event replay used for */ws/orders* also applies here. Position data will be replayed from the offset defined during WebSocket disconnection.
  - Contains 2 Kafka consumers
    1. Listens to *orders_updated* topic. Data is sent to */ws/orders*, populating respective grid on the frontend. 
    2. Listens to *positions_updated* topic. Data is sent to */ws/orders*, populating respective grid on the frontend.
3. **order_resolver**
  - Serves as the workload for sending requests to external exchange APIs. 
  - Evaluates any relevant information returned from external APIs that should trigger the update to Redis caches. 
  - Sends relevant position & order data to the *orders_updated* and *orders_executed* topics which feeds the WebSocket connections that the frontend is connected to.
4. **frontend**
5. **Kafka**
6. **Redis** 




## Conclusion


Stay tuned for future blog posts, where I'll dive deeper into the technical details of the project, share my experiences with various containerized applications, and discuss the lessons I've learned along the way.

Thank you for reading!
