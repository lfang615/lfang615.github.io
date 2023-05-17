---
layout: post
title: "Trade on Bybit without the VPN hassle"
date: 2023-04-11
categories: kubernetes containers devops distributed kafka redis
thumbnail-img: ""
js:
  - https://cdn.jsdelivr.net/npm/mermaid@8.13.5/dist/mermaid.min.js
---

## Motivations

When taking upon an interest in capital markets, you'll sooner or later bear the burden of navigating the regulations that govern these markets. Then if your curiousities ever lead you to browse around and compare financial services offered in comporable developed first world nations, it'll become apparent (*not much analytical skills needed*) that the U.S. is much more strict as to how they allow their residents to partipate in the markets.

For those among the crypto crowd in the U.S. researching crypto exchanges offering derivitive products, you may have narrowed in on Bybit as a suitable candidate for your trading activities, since their KYC policies are more liberal compared to the other players in the space. They provide 2 separate platforms, one for U.S. residents and another for everyone else abroad. For those with a little technical know how, I'm sure you can figure out how to access the latter (*tip* there's a clue in the title of the post).

Using the method above pigeon holes you to their web app to make trades which is only made available in the desktop version of their platform, making usability a nightmare. Mobile VPN connections, in my experience can't be relied on to provide stable connections (*from my provider at least*) for long durations which causes the site the session to become invalid and ultimately creates an annoying loop of reconnecting to VPN server, inputing login details, fetching the multifactor code from Google Authenticator, every time I want to place a trade when I'm away from the a desktop.

This project was started to resolve those annoyances and to explore various tools used in distributed systems

[Project Repo](https://github.com/lfang615/bybit-service)

## Description
 
- Motivations were described above, but the primary utility was to have a self hosted layer I can use to interact with Bybit.
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


## _Quick overview of the services responsibilities_

*Tip* - It occurred to me after I decided to create a mermaid diagram , but for those using ChatGPT-4 (intended audience) through the chat interface you've probably realized they've gated its multimodal capabilities by preventing you from uploading images through the chat interface. Mermaid syntax can be a way around it, at least as far as diagrams are concerned. By using mermaid syntax to visually communicate your project to ChatGPT-4 along with some crafty prompt engineering you can bootrap a project VERY quickly.

### Overview of each service and their responsibility

1. order_manager
  - 
3. order_resolver
4. Kafka
5. **Redis 




## Conclusion


Stay tuned for future blog posts, where I'll dive deeper into the technical details of the project, share my experiences with various containerized applications, and discuss the lessons I've learned along the way.

Thank you for reading!
