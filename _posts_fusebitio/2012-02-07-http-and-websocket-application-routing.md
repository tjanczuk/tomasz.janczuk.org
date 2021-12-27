---
tags: ['post']
post_og_image: 'site'
date: '2012-02-07'  
post_title: HTTP and WebSocket application routing using arrjs reduces the cost
  of hosting applications
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: http-and-websocket-application-routing
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




Today I started [arrjs](https://github.com/tjanczuk/arrjs), a system that supports HTTP and WebSocket request routing in server farms. It allows hosting several HTTP or WebSocket applications in a single server farm while exposing all endpoints over a single TCP port. The [arrjs GitHub project page](https://github.com/tjanczuk/arrjs) explains the benefits and features of arrjs in more detail; key features include hostname based routing, HTTP/HTTPS and WS/WSS support and support for application-specific X.509 certificates for SSL using SNI.   

The key benefit of arrjs is lowering the cost of running multiple HTTP or WebSocket applications by reducing the amount of hardware necessary to run them.   

When planning deployment of a web application, the number and specification of servers must allow for handling the expected average load as well as spikes in that load. That means that part of the computing capacity is idle between traffic spikes, which increases the cost of hosting the application. Consider two applications: app1 and app2 are deployed on 4 servers each, 2 of which are used under average load and remaining 2 only utilized during traffic spikes:  

 ![Two web applications is separate farms](http://lh5.ggpht.com/-nyuE3pHDsj8/TzHMazCYMcI/AAAAAAAAB7g/yhoW7PYt8yI/Screen-Shot-2012-02-07-at-4.39.37-PM.png?imgmax=800)  

Assuming each of the servers costs $20/month to operate, the total cost of operating these two applications in the configuration above is $160/month.   

If the traffic spikes of app1 and app2 do not coincide in time (e.g. app1 is a store specializing in Christmas schmuck while app2 sells Super Bowl apparel), there is an opportunity to reduce the total number of servers required to operate both applications by combining the server capacity dedicated to handling traffic spikes in a single web farm:   

 ![Two web applications in a single farm using arrjs](http://lh5.ggpht.com/-8T5MNfGo4Ho/TzHMbPCDbXI/AAAAAAAAB7w/Qx37Y7NmdKI/Screen-Shot-2012-02-07-at-4.52.05-PM%25255B2%25255D.png?imgmax=800)  

Running app1 and app2 in this configuration costs $120/month, a 25% reduction in costs compared to previous configuration. The effect becomes even more pronounced with the economies of scale associated with running a larger number of applications in a single farm of servers.   

Arrjs helps you run multiple web applications in a single farm of servers to realize these cost reductions.   

Arrjs is a system that supports running multiple web applications in a single web farm while ensuring they are all externally accessible using a single port 80 for HTTP/WebSocket traffic and a single port 443 for HTTPS/secure WebSocket traffic. Arrjs is a reverse HTTP proxy capable of message-based activation of applications and keeping track of a dynamic routing table for routing requests targeting disparate applications. Arrjs is implemented in node.js and uses MongoDB, and can therefore be deployed on Windows, *nix, or MacOS.   

Read more at [https://github.com/tjanczuk/arrjs](https://github.com/tjanczuk/arrjs).   