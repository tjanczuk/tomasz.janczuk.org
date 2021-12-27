---
tags: ['post']
post_og_image: 'site'
date: '2010-07-11'  
post_title: "Laharsub: HTTP REST pub/sub service for AJAX and Silverlight web clients"
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: laharsub-http-rest-pubsub-service-for
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




Today I would like to announce the creation of the [Laharsub](http://laharsub.codeplex.com) project on Codeplex and invite you to download, use, review, comment, discuss, share ideas, and otherwise collaborate on this interesting technology.   

The objectives and scope of the Laharsub project are captured [on the project’s site](http://laharsub.codeplex.com); to quote:  



### Overview        
The growing trend of application development targeting web clients and open standards (HTML, JavaScript, HTML 5) generates demand for ever richer experiences for the web platform. Many interesting web applications require more than the traditional request/response communication pattern enabled by Ajax technologies. Web based chat or collaboration applications, multiplayer online gaming, or news or financial data dissemination applications require low latency, asynchronous data transfer from the server to the web client. Communication patterns in most of these applications are using a pub/sub model, with topics that multiple clients can publish to, subscribe to, and receive notifications from.        
        


### Goals        
The goal of the Laharsub project is to provide a solution that makes it easy for web applications to organize internet scale message exchange using a publish/subscribe pattern. The project is an ongoing experiment with a variety of web technologies. Current focus is on AJAX (in particular jQuery) and Silverlight clients, a REST based HTTP long polling subscription protocol implemented by a .NET WCF HTTP middle tier service, and researching middle tier and back end technologies that enable scale-out to a large number of clients.  

The project is directly related to my interest in rich communication patterns for web application. It draws on the lessons learned from the HTTP Polling Duplex protocol in Silverlight, and is an attempt to experiment with alternative approaches that may improve developer’s experience in a number of scenarios of data push to a web client. In particular:  

* It focuses on backend scalability,  
* It focuses on Ajax clients as much as on RIA (Silverlight) clients,  
* It focuses on the publish/subscribe message exchange pattern as opposed to duplex messaging capability, reflecting the majority use case for the HTTP Polling Duplex protocol in Silverlight.  
  

I am looking forward to your feedback.  