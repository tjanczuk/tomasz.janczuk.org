---
post_title: How to save $5 million running a Node.js application
date: 2013-05-25
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: how-to-save-$5-million-running-a-node.js-application
post_date_in_url: true
post_og_image: site
post_excerpt: Tomek on Software - shaken, not stirred
tags:
  - post
---




### Prerequisites  

Before you can save $5 million, you must be spending more than that. One way to spend over $5 million is to run a large Node.js web application. For example, an application that requires 10,000 CPU cores to handle its traffic. For the purpose of this napkin math, let’s assume this Node.js application is deployed on 1,250 extra large (8 core CPU) Windows VMs in Windows Azure at an annual cost of $6,912,000 (1250 x 12 x $460.80):  

 ![image](http://lh6.ggpht.com/-wZNafhKhrio/UaEKmZTT-FI/AAAAAAAADeQ/8BkzPVQtAAg/image_thumb%25255B1%25255D.png?imgmax=800)   

### The web application  

With the problem of spending more than $5 million out of the way, let’s talk about the application itself. There is a class of web applications for which horizontal data partitioning is the most sensible approach of scaling out. Many relay applications are in this class. Imagine a web chat application, which allows several browser clients to connect to a chat room on the server to exchange data in real time. There may be millions of chat rooms handled by the web application at a time, which requires a total of 10,000 CPU cores. However, individual chat rooms typically have only a handful of participants and does not consume a lot of resources. Given that we can horizontally partition the chat rooms such that all calls to a particular chat room are always handled by a specific process on a one of the servers. This allows us to keep chat room state in-process and therefore improve the response latency compared to an approach with externalized state.   

The Node.js process is single threaded, so a single process can only fully utilize one CPU core. On a VM with 8 CPU cores, running 8 Node.js processes will ensure that the overall CPU is fully utilized.   

To summarize, the deployment of the application requires 1,250 extra large (8 CPU core) servers to accommodate the traffic. Each server runs 8 Node.js processes. Each Node.js process handles many chat rooms:  

 ![image](http://lh5.ggpht.com/-YRkuYBvpoUc/UaEKnKx3LJI/AAAAAAAADeg/hK8TOIIp5po/image_thumb%25255B4%25255D.png?imgmax=800)   

To support horizontal partitioning the system must have a routing logic in place that knows how to route all requests targeting a specific chat room to the Node.js process that keeps the chat room’s state. To build such routing system, individual chat rooms must be addressable, for example using HTTP URL path segments as follows:  

{% highlight javascript linenos %}
   http://megachat.com/{server_id}/{process_id}/{chatroom_id}
  

{% endhighlight %}



or a combination of a DNS name and HTTP URL path segment:

{% highlight javascript linenos %}
http://{server_id}.megachat.com/{process_id}/{chatroom_id}
  

{% endhighlight %}



In this post I am going to assume that the problem of server level routing had already been solved: when the system receives an HTTP or WebSocket request from the client, a router consistently routes the request to the server identified with the *{server_id}*. Once a request is received by the appropriate server, additional routing mechanism is necessary to dispatch that request to the Node.js process identified by the *{process_id}* URL segment. And a good way of doing that is the 5 million dollar question this post is about. 

### The natural choice: Application Request Routing

When a request of the form *http://megachat.com/{process_id}/{chatroom_id}* is received by a server, there must be a mechanism in place to route it to a particular Node.js process based on the *{process_id}* segment of the URL. This is typically accomplished with an HTTP reverse proxy that routes requests to other processes that listen on distinct TCP ports, for example:

 ![image](http://lh3.ggpht.com/-wsJ1PHIz23U/UaEKnuxXv8I/AAAAAAAADe0/O_nCuVivLQI/image_thumb%25255B7%25255D.png?imgmax=800) 



In the example above, the *{process_id}* segment of the URL is an ordinal number of a Node.js process running on the server, an integer between 1 and 8. Given that value, the reverse proxy chooses to forward the request to a TCP port number in the range 8081-8088. 

There are many HTTP reverse proxy technologies that allow this sort of configuration, from Nginx, to Application Request Routing (ARR) in Internet Information Services (IIS), to several solutions built with Node.js (e.g. node-http-proxy). Since the application is running on Windows, the natural choice would be to use Application Request Routing in IIS. Let’s assume ARR is what the web application is using currently. 

### The alternative: using HTTP.SYS port sharing to implement HTTP reverse proxy logic

Part of the Windows operating system is a kernel level HTTP stack called HTTP.SYS. One of the many interesting features of HTTP.SYS is port sharing. With port sharing, several processes on the machine can register HTTP listeners on the same TCP port but distinct URL path segments. For example, one process can listen for all messages sent to http://{server}:80/foo (and subordinate URLs), while another process can listen for all messages sent to http://{server}:80/bar (and subordinate URLs). This mechanism would allow the HTTP reverse proxy logic to be implemented at the kernel level as long as we can access HTTP.SYS functionality from Node.js:

 ![image](http://lh5.ggpht.com/-32b92JUYvks/UaEKocbSiBI/AAAAAAAADfE/Ua3Bmeq2AiM/image_thumb%25255B10%25255D.png?imgmax=800) 

The *httpsys* module enables use of HTTP.SYS in Node.js applications running on Windows. With the *httpsys* module an application can register an HTTP listener directly with HTTP.SYS and use the kernel mode HTTP stack on Windows instead of the user mode HTTP stack that Node.js ships with. The *httpsys* module preserves the server side HTTP APIs of Node.js, so minimal changes are required in your application code when switching from the built-in HTTP stack to HTTP.SYS. Here is a *Hello, world* application that uses the *httpsys* module:

 ![image](http://lh4.ggpht.com/-XbZ0x2XTS-U/UaEKo62XEVI/AAAAAAAADfU/C16sV0ko9q4/image_thumb%25255B13%25255D.png?imgmax=800) 

Compared to a standard *Hello, world* sample for Node.js, there are two differences. In line 1, the *httpsys* module is used instead of the built-in *http* module. In line 6, the HTTP server starts listening on a [URL prefix string](http://msdn.microsoft.com/en-us/library/windows/desktop/aa364698(v=vs.85).aspx) instead of TCP port number. The server in this example will only receive HTTP requests that arrive on port 80 *and* whose first segment of the URL is equal to /1/. You can read more about the port sharing feature of *httpsys* module [here](https://github.com/tjanczuk/httpsys#port-sharing). 

### Performance

Implementing the HTTP reverse proxy logic required for horizontal partitioning using kernel level port sharing with HTTP.SYS offers much better performance than doing the same using Application Request Routing in IIS. In addition, [the raw performance of HTTP.SYS is superior to the performance of the HTTP stack in Node.js](http://tomasz.janczuk.org/2012/08/the-httpsys-stack-for-nodejs-apps-on.html). 

To demonstrate this, I measured and compared the throughput of the ARR and HTTP.SYS solution. The measurement was done as follows:

* The server machine was an 8-core Intel Xenon W3550 @ 3GHz  
* In the ARR variant, I created 8 Node.js processes listening on ports 8081-8088 using the built-in Node.js HTTP stack and configured ARR on port 8080 to reverse proxy incoming requests across these ports based on the value of the first segment of the URL path of the request.  
* In the HTTP.SYS variant, I created 8 Node.js processes using the *httpsys* module to listen on URL prefix strings of the form *http://*:8080/{n}* where *n* was an integer between 1 and 8.  
* The Node.js applications were returning a simple *Hello, world* response to all requests.  
* I used 2 client machines running [WCAT](http://www.iis.net/downloads/community/2007/05/wcat-63-(x86)) to achieve 100% CPU utilization of the server during the measurement to ensure the results are normalized. Each measurement had a 30 second warm-up period followed by 30 second period of measurement.  
* WCAT was configured to issue requests evenly distributed across the eight Node.js processes.  


Here are the results:

 ![image](http://lh6.ggpht.com/-LfqdwxkWNjE/UaEKplsGxXI/AAAAAAAADfk/edNyrVHUT8o/image_thumb%25255B16%25255D.png?imgmax=800) 

It looks like kernel level port sharing with HTTP.SYS offers 6.1x better performance than ARR with IIS when used as a mechanism to implement a reverse proxy logic in this measurement. Time to go back to the napkin math and convert this to dollars. 

### Show me the money: saving the $5 million

By replacing the ARR/IIS reverse proxy mechanism in our web application with one based on HTTP.SYS port sharing, we can increase the capacity of the application by the factor of 6.1. Conversely, we can maintain the current capacity of the web application and reduce the deployment size by the factor of 6.1. Instead of using 1,250 extra large VMs we started with, we can now use 205 machines and still serve the same traffic. This translates to a drastic reduction in annual infrastructure cost:

 ![image](http://lh6.ggpht.com/-IfgQSdNQRCQ/UaEKqTGj_LI/AAAAAAAADf0/H6n9BUWrzlA/image_thumb%25255B19%25255D.png?imgmax=800) 

Switching from ARR/IIS to HTTP.SYS port sharing saved $5,778,432. It is somehow larger saving than the $5 million originally promised by this post. You could spend the extra $778,432 in savings on a [Bavaria Cruiser 56](http://www.bavariayachts.com/bavaria-cruiser-56.php) with still enough change left to take you comfortably on a cruise around the world. If you do, please send me a post card.   }