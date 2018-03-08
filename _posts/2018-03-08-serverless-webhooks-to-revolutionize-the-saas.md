---
layout: post
title: Serverless webhooks to revolutionize the SaaS
author: Tomasz Janczuk
tags:
- serverless
- webtask
- webtasks
- auth0
- nodejs
- SaaS
- webhook
- javascript
---

In 2007, Jeff Lindsay introduced *webhooks* in his [Webhooks to revolutionize the web](http://progrium.com/blog/2007/05/03/web-hooks-to-revolutionize-the-web/) article. Since then, webhooks have become the dominant mechanism for exposing events from web services. Fast forward to 2017, and a new, better pattern started to emerge: serverless webhooks. 

### webhooks (2007)

This is how Jeff defined webhooks in his iconic article from 2007: 

> Webhooks are essentially user defined callbacks made with HTTP POST. To support webhooks, you allow the user to specify a URL where your application will post to and on what events. Now your application is pushing data out wherever your users want.

This is a very elegant and powerful concept. It allows for a clean, protocol-based decoupling of the system that exposes an event from the system that processes that event. It is flexible enough to allow for a wide range of scenarios, including system integration, data transformation, or customization of behavior. It also embraces the asynchronous nature of interactions between web services. 

<img src="/assets/post_images/2018-03-07/0.png" class="tj-img-diagram-100" alt="2007 webhooks">

Using webhooks requires setting up and running another web service. Servers, devops, monitoring, SSL, uptime, failover strategy, etc. For years this has been a small price to pay given the benefits webhooks offered, and the lack of a better alternative. However, given cloud computing advances over the last decade, there is now an opportunity to further improve on this core concept. 

### Serverless webhooks (2017)

Fast forward from 2007 to 2017, through IaaS, PaaS, all the way to FaaS (Function-as-a-Service, aka *serverless*). The essence of the serverless computing paradigm is to organize your logic into small units of computation you can refer to as *functions*. A single function maps very well to a single webhook.

Reacting to web service events with arbitrary business logic is the core value of webhooks. Imagine an experience where users of a SaaS platform can focus on implementing this business logic as a *function* without having to run a service to expose it as an HTTP endpoint:  

<img src="/assets/post_images/2018-03-07/1.png" class="tj-img-diagram-100" alt="2017 Serverless webhooks">

There are several immediate advantages such a serverless webhook model brings for a SaaS platform compared to the original webhooks:

1. **The users** of a SaaS platform benefit from a dramatically improved experience of implementing custom event logic and the reduction of time to market: users can focus on the business logic without dealing with issues related to running a web service.  
2. **The sales engineers** of a SaaS company are enabled to rapidly prototype custom integrations with little effort, shortening the sales cycles and highlighting the power of the platform.  
3. **The support team** of a SaaS organization is equipped to better serve the customers since they have full insight into the implementation logic of the function handling the event.   
4. Last but not least, **the SaaS platform itself** is using cutting-edge extensibility technology based on the latest cloud computing trends, which sets it apart from the competition.  

Ten years after the introduction of the webhook concept, [this is what Jeff Lindsay says about the serverless webhook pattern](https://twitter.com/progrium/status/864588610858881029):

<img src="/assets/post_images/2018-03-07/2.png" class="tj-img-diagram-100" alt="This was the whole point of webhooks">

There are many existing implementations of this serverless webhook paradigm in practice. The [Twilio Functions](https://www.twilio.com/functions) and [Salesforce Apex](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_dev_guide.htm) are good examples. Until recently, these advanced solutions have been reserved for the SaaS platforms run by sophisticated, technology-savvy organizations. 

### Serverless webhooks for your SaaS

If you are operating a SaaS platform that could benefit from serverless webhooks, you may ask yourself what it takes to implement them. In a subsequent post, I will describe an architectural blueprint for building a serverless webhook solution for your own SaaS platform, based on the lessons learned from the [Auth0 Extend](https://auth0.com/extend) product in Auth0.   
