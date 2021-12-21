---
tags: ['post']
post_og_image: 'site'
date: '2012-06-13'  
post_title: Performance of hosting node.js applications in iisnode vs
  self-hosting on Windows
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: performance-of-hosting-node.js-applications-in-iisnode-vs-self-hosting-on-windows
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




In this post I am sharing high level, general observations related to performance implications of choosing one of the two major ways of hosting node.js HTTP applications on Windows.   

Node.js HTTP applications can be run in two ways on Windows today:  

* by self-hosting the application using the [node.exe process](http://nodejs.org/#download) directly on top of a TCP socket (similarly to the most common way of running node.js applications on other operating systems),  
* by hosting the application in the Internet Information Services (IIS) web server using the [iisnode](https://github.com/tjanczuk/iisnode) module.  
  

Besides performance, there are other considerations for choosing one method or the other. You may want to review [some of the benefits of hosting node.js applications in IIS](http://tomasz.janczuk.org/2011/08/hosting-nodejs-applications-in-iis-on.html). If you intend to host your node.js application in Windows Azure, make sure to also read up on the [three ways of hosting node.js applications in Windows Azure](http://tomasz.janczuk.org/2012/06/three-ways-of-deploying-nodejs-apps-to.html).   

### Four reasons you should distrust this post (aka disclaimer)  

1. The measurements were taken using node.js v0.6.6 and iisnode v0.1.13, and both of these technologies continue to rapidly move forward. It is likely that the same measurements done on the most current versions would yield different results.  
2. The scenarios measured may not represent what your application is doing. There is a rather dramatic difference in the results depending on the scenario below, and your actual app is unlikely to closely match any of these scenarios.  
3. I have developed [iisnode](https://github.com/tjanczuk/iisnode), why would you trust me to objectively talk about its performance?  
4. Buts. The area of performance measurements yields itself very well to “but”-type of arguments. But the scenario is not real. But the machine is not like a machine I like better. But the payload is too small/large. You get the idea. So in the spirit of transparency [you can see the test code](https://github.com/tjanczuk/iisnode/tree/master/test/performance), you can repeat the measurements, you can add your own tests (and make a pull request) if you think you have a useful scenario to measure.  
  

### Results  

The table below shows comparison of throughput between self-hosted node.exe and IIS+iisnode+node.exe:  

 ![Screen Shot 2012-06-13 at 3.03.43 PM](http://lh3.ggpht.com/-fIIjSarL_RA/T9kUCYXgDRI/AAAAAAAACCs/ADl94P-2xhA/Screen%252520Shot%2525202012-06-13%252520at%2525203.03.43%252520PM_thumb%25255B2%25255D.png?imgmax=800)  

Unless otherwise noted below, all measurements were taken at server CPU utilization > 90%. The configuration involved one quad core Windows 2008 server with node.js v0.6.6 and iisnode v0.1.13. Depending on scenario, one or two WCAT client machines were required to saturate the server. Clients and server were connected to the same switch but were not on private network.     
  
[1] [See test code](https://github.com/tjanczuk/iisnode/tree/master/test/performance/www/default). In this scenario node.js application returns a simple “hello, world” plaintext response to an HTTP GET request. This is the only scenario that uses a single node.exe instance (both self-hosted and IIS hosted). Given the single-threaded nature of node.exe, the multi-core server CPU is never fully utilized. However, the node.exe process has consistently utilized > 23% of CPU, which corresponds to almost full utilization of a single core of the quad-core server.      
  
[1.1] Although in this scenario iisnode achieved better throughput than self-hosted node.exe, it was accomplished with almost twice as large CPU utilization (~60% in case of iisnode compared to ~30% in case of node.exe).      
  
[2] [See test code](https://github.com/tjanczuk/iisnode/tree/master/test/performance/www/cluster). This scenario uses the same node.js “hello world” application as scenario #1, but it utilizes 4 node.exe processes to saturate the CPU on the quad core server. In case of self-hosting node.exe, the built-in cluster functionality is employed. In case of iisnode, the built-in capability to spawn and route to multiple node.exe processes per application is used.      
  
[3] [See test code](https://github.com/tjanczuk/iisnode/tree/master/test/performance/www/express). This scenario simulates using node.js to serve a web site using express - one of the most popular MVC frameworks for node. The web site consists of an HTTP endpoint that dynamically generates HTML that displays a randomly generated order information using the jade rendering engine. The HTML refers to a static CSS file. As part of each WCAT transaction, one request is made to the HTML endpoint that generates dynamic content and the another to get the static CSS file. In case of self-hosted node.js application, the static file is served by the express framework using the connect.static middleware (which is built into express and a most likely mechanism to be used by someone serving static content from express). In case of IIS, requests for static content are actually not handled by iisnode at all – a URL rewrite module causes that request to be handled by the native static file handler (which plays to the “side by side with other content types” benefits of iisnode).      
  
[4] [See test code](https://github.com/tjanczuk/iisnode/tree/master/test/performance/www/express-dynamic). This scenario simulates using node.js to implement web APIs: HTTP endpoints that generate dynamic content. In this particular case we are reusing the order generation and HTML rendering endpoint from scenario #3, without including a request for static CSS file in the WCAT transaction.    

### Interpretation  

#### IIS+iisnode does nicely in a mixed content scenario (Web site)  

Scenario #3 represents an end to end web site with node.js using the express MVC framework: some of the content is dynamically generated (dynamic HTML generated from a model), and some is static (CSS, potentially images). Serving static content from node.js is slow enough to more than offset any IIS+iisnode overhead in the dynamic content generation part of the application, resulting in the overall IIS+iisnode throughput 305% larger than self-hosting node.exe. This scenario is a good fit for IIS+iisnode as combines the most efficient aspects of both technologies into one convenient package.  

#### IIS rocks with serving of static content  

As scenario #3 shows, IIS+iisnode beats self-hosted node.js in serving static content by a very large factor. It is to be expected that a native file handler implementation would perform much better than any implementation in a managed environment like node.js. The kinds of applications that benefit from this are applications that serve static content as a substantial portion of their transactions.   

#### IIS+iisnode add substantial overhead in dynamic content scenarios (Web APIs)  

Scenarios #2 and #4 demonstrate that IIS+iisnode introduce substantial CPU overhead in processing request that require dynamic content generation in a node.js application. These scenarios represent a class of “Web APIs” – HTTP endpoints that are geared towards dynamic content generation, e.g. JSON, XML, ATOM, RSS, but also dynamic HTML.     
      
As the amount of CPU intensive work required to generate the response in the application code increases, the IIS+iisnode overhead becomes less pronounced. On one extreme of CPU intensity we have scenario #2, with extremely lightweight application logic. In that case IIS+iisnode overhead is over 100%, resulting in throughput degradation to 49% compared to self-hosted node.exe. Scenario #4 is a reasonable approximation of the CPU intensity of an MVC application that applies some business logic to the model before rendering the results to the client. This scenario shows smaller IIS+iisnode overhead with throughput reduced to 73% of self-hosted node.exe.  }