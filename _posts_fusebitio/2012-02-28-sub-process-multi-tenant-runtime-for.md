---
post_title: Sub-process, multi-tenant runtime for HTTP APIs using node.js
date: 2012-02-28
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: sub-process,-multi-tenant-runtime-for-http-apis-using-node.js
post_date_in_url: true
post_og_image: site
post_excerpt: Tomek on Software - shaken, not stirred
tags:
  - post
---




Lately I have been working on [haiku-http](http://tjanczuk.github.com/haiku-http/): an experiment to create a node.js-based, multi-tenant runtime for hosting HTTP APIs within a single process. The key objective of the project is to find out if we can reduce the cost of hosting HTTP APIs written in node.js by securely and reliably sharing resources of a single process across multiple tenants.   

Haiku-http is a different approach to the problem of sub-process multi-tenancy I have already worked on in the [denser](https://github.com/tjanczuk/denser) project. I presented the denser project at [JSConf.EU 2011 in Berlin](https://github.com/downloads/tjanczuk/denser/High%20density%20server%20side%20JavaScript.pdf); here is the visual summary by [@annalena](https://twitter.com/#!/annalena):  

 ![image](http://lh6.ggpht.com/-8YM4DFc7gfY/T00OeiuyH2I/AAAAAAAAB8I/VfcIFP_aYw8/image_thumb%25255B2%25255D.png?imgmax=800)  

In the denser project I created a sub-process, multi-tenant environment using the V8 JavaScript engine and providing a simple node.js-like programming model on top of it. It was different from node.js in its use of V8 isolates, each running on its own OS thread. It also offered a mode of running applications in a single V8 isolate and one thread using a separate V8 context for each application, similar to what node.js can do using the vm module.   

Unlike denser, haiku-http builds entirely on top of node.js. It currently does not require any changes in the node.js runtime, nor any native extensions. The key advantage of this model is a programming environment that leverages the rich ecosystem of modules in and around of node.js, as well as capability to run it cross-platform.   

The rest of the post relates to node.js v0.7.2.    

### Application density and the cost of hosting web applications  

Many web applications created today are deployed in shared hosting environments. The cost of hosting these applications incurred by the hosting company is the primary factor affecting the price of the hosting plan offered to end users.   

Many hosting companies support a range of hosting plans: from empty rack space in a data center, to managed physical machines, to virtual machines, and finally to individual OS processes (the plans that do not support “root level” access). The price of a hosting plan decreases as more web applications from different tenants share the same set of resources in a data center, with managed machines costing tens or hundreds of dollars per month all the way down to the least expensive shared hosting plans that cost a few bucks.  

 ![image](http://lh3.ggpht.com/-5ndfzLSaWU0/T00OfT64LyI/AAAAAAAAB8Y/ykzQauOYuJc/image_thumb7.png?imgmax=800)  

The hypothesis behind haiku-http is that by sharing the resources of a single OS process (memory, handles, threads) across multiple applications we can run more applications on a single machine, and therefore reduce the marginal cost of hosting such applications compared to dedicating an entire OS process to each of them.   

Prior measurements I have done with the [denser](https://github.com/tjanczuk/denser) project suggest it is possible to achieve a 15x increase of density compared to process-level hosting for a certain class of applications. That means **hosting costs of $7 at sub-process density compared to $100 for hosting the same workload at process density**. Such economy of scale is only possible for a very specific kind of applications: applications that are mostly waiting for stimulus without consuming a lot of resources (memory, bandwidth, CPU). In the measurement I was running 1000 isolated instances of a an HTTP long polling based web chat application, which spends most of its time waiting for IO completions.   

### The haiku-http approach  

The haiku-http project provides a node.js-based runtime for multiple applications using a single node.js process. Below are key design choices I made for haiku-http:  

#### Application === HTTP request handler

Haiku-http supports running JavaScript code that accepts an HTTP request and generates an HTTP response. Writing a haiku-http application is conceptually similar to implementing the body of the node.js HTTP request handler. For example, given the following node.js application:      

{% highlight javascript linenos %}
       require('http').createServer(function(req, res) {  
    res.writeHead(200)  
    res.end('Hello, world!')  
}).listen(8000)
  

{% endhighlight %}


a corresponding haiku-http handler would be:

      
  


{% highlight javascript linenos %}
res.writeHead(200)  
res.end('Hello, world!')
  
In addition to the req and res globals, one can use a subset of node.js modules described in more detail in the section about sandboxing below. 

        

{% endhighlight %}



#### Every HTTP request is individually sandboxed 

        
  
One HTTP handler cannot access in-memory state of another HTTP handler. Also, there is no sharing of in-memory data between consecutive invocations of the same HTTP handler (persistent data must be stored outside of the address space of the handler, e.g. using HTTP cookies or a backend database). In addition to the in-memory state explicitly created by the handler, no node.js modules are shared between handlers. 

        


#### One node.js process handles multiple HTTP requests  
  
The node.js process runs code that establishes the listener and then delegates handling of an HTTP request to a haiku-http handler after creating a sandboxed execution environment for that handler. 

        


#### Node.js cluster is used for scale and risk management  
  
Haiku-http runs a number of child processes in a cluster. This helps fully utilize the CPU on multi-processor machines but is also useful in limiting the damage of an attack that causes a worker process crash. More about this in the section about local denial of service attacks below.



### Programming and execution model

You can read more about the details of the programing model and the execution model as well as play with a sample deployment of haiku-http [here](https://tjanczuk.github.com/haiku-http). 

The gist of the idea is that you can issue any HTTP request to an HTTP/S endpoint exposed by the haiku-http service and provide the URL of the haiku handler that should handle that request using the x-haiku-handler URL query parameter, e.g.: 

{% highlight text linenos %}
http://haiku.cloudapp.net/?x-haiku-handler=https://raw.github.com/gist/1848111

{% endhighlight %}



The [http://haiku.cloudapp.net](http://haiku.cloudapp.net) is a sample deployment of the haiku-http service in [Windows Azure](https://www.windowsazure.com/en-us/). (The endpoint may be down when you try it, but you can easily set up your own anywhere node.js runs using [http-haiku project](https://github.com/tjanczuk/haiku-http).)

### Sandboxing

One of the key requirements for a multi-tenant environment is that one tenant’s data cannot be accessed or modified by another tenant. Haiku-http achieves data isolation with the following mechanisms:

#### Transient data is in memory, persistent data outside of the machine  
  
Every haiku-http handler can create transient, per-request in-memory state. Haiku-http runtime creates a new V8 context with its own global object for every HTTP request, so that in-memory data is not accessible outside of that handler code. In fact, even subsequent execution of the same handler cannot access any in-memory data created by previous or concurrent executions of the same handler. 

      
  
This forces the application to persistently store any state transcending the lifetime of a single HTTP request using mechanism other than than managed memory. (This is a good design practice anyway to allow for HTTP server code to scale out; in-memory state should not be used for anything other than performance optimization). In the current version of haiku-http, application can store persistent data in a MongoDB database or roundtrip it to the client using HTTP cookies.    
  
The goal of this approach to state management is to ensure that no application data is protected using the OS security mechanisms specific to the node.js process that runs code of multiple tenants, which would allow access to that data to all handlers. Specifically, file system access is explicitly not supported. 

      


#### No shared node.js modules between handlers 

      
  
Normally the node.js module system caches node.js modules at the process level. In the context of haiku-http this situation creates an isolation problem, since one handler can now affect the behavior of another handler. For example, one handler could wrap the implementation of the http.connect method with the intent of intercepting requests issued by other handlers. 

      
  
In haiku-http, node.js module caching is refactored such that no module instance is shared between handlers. Module caching is only enabled for the duration of a single ‘require episode’: a synchronous code path starting with a topmost call the the ‘require’ method on a call stack which may recursively call ‘require’ again. Such scoping of the module cache enables cyclic module dependencies while ensuring that a different module instance is returned for each ‘require episode’. 

      


#### Outgoing networking only. No listeners 

      
  
Handler code can make outgoing network calls (using HTTP/S and TCP/TLS as well as Mongo in the current version), but it cannot establish listeners. Listening functionality is not needed since the only entry point to the haiku code is through the HTTP listener already established by the haiku runtime. Moreover, access to the server side functionality would enable handler code to intercept calls made to other handlers, therefore breaching the data isolation boundary.

      


#### Subset of node.js functionality. 

      
  
To achieve the sandboxing behavior, haiku-http offers a subset of node.js functionality. It is done by rejecting requests for specific node.js modules (e.g. ‘fs’), and wrapping other node.js modules to remove certain APIs (e.g. http.createServer) or augment their behavior. In the current version, a haiku-http handler can use the client side functionality of the http, https, and net modules, use the request module, as well as perform MongoDB operations using the mongodb module. 

      
  
In addition to limiting the API surface area available to a haiku-http handler, application code is evaluated in [ECMAScript 5 strict mode](https://developer.mozilla.org/en/JavaScript/Strict_mode) to prevent handler code from accessing the trusted part of the system by dereferencing objects on the call stack. 



### Local denial of service attacks

Local denial of service attacks are attacks where one handler causes inordinate resource consumption, therefore preventing other handlers running on the same machine or process from completing their job. 

Current set of features offered by node.js and V8 does not support prevention of the full range of local denial of service attacks. Haiku-http therefore focuses on detection and recovery rather than prevention. At the same time, some new features planned for node.js v0.8 (in particular [Domains](http://groups.google.com/group/nodejs/browse_thread/thread/79504e62223f3bf0)) potentially offer an opportunity to improve prevention capabilities (or reduce the effects of an attack). 

The key principle behind haiku-http model for attack recovery is limiting destruction to one worker process at a time. Haiku-http uses node.js cluster to run a configurable number of worker processes. They are used not only for scale out to multi-processor machines, but also for limiting the maximum damage a single misbehaving handler can do to the haiku-http system. The worst case scenario is that a evil handler will lead to the crash of the worker process and abnormal termination of all HTTP requests active in that process at that time. The haiku-http runtime will subsequently replace the crashed process with a new one. 

With this basic principle in mind, the attack detection and recovery approach of haiku-http is best explained with a few examples of local denial of service attacks. 

#### Unhandled exceptions

If a regular node.js application throws an unhandled exception, the best practice is to terminate the node.js process. This is because there is currently (v0.7.2) no way to reason about and release the native resources that might have been allocated and not freed by the code that generated the exception. If the exception was ignored, chances are native resource leaks would grow over time. 

So assuming the haiku-http handler as follows:

{% highlight javascript linenos %}
res.writeHead(200)  
throw new Error('An exception')  
res.end('Hello, world!')
  

{% endhighlight %}



the haiku-http runtime will intercept the unhandled exception and proceed to gracefully terminate the process. It will stop accepting new HTTP requests and wait for a configurable timeout for the currently active requests to drain. It will then kill the process. The master process in the cluster will detect process crash and will create a new worker process to replace it. 

#### Blocking operations

The all-time favorite way of hanging node.js runtime is to perform a blocking operation like this:

{% highlight javascript linenos %}
while(true)
  

{% endhighlight %}



This single line will effectively block the event loop in a single-threaded node.js runtime and prevent the process from accepting new HTTP requests as well as doing any work on behalf of other HTTP requests active in that process at the time. There is currently no way in node.js to prevent it, but haiku-http implements a mechanism to detect runaway processes like this one. The haiku-http master process is periodically issuing a challenge to all child processes in the cluster using IPC mechanisms. If the child does not respond to the challenge within the preconfigured timeout, the master process assumes the child’s event loop is blocked and proceeds to terminate the child and replace it with a new instance. Note that in this case, unlike in the case of an unhandled exception, all HTTP requests active in the child process are abnormally terminated. 

#### Very long running handlers

Some handlers may be consuming more then their fair share of CPU and system resources. Node.js runtime or V8 do not currently provide a good way to measure CPU and memory consumption at the granularity of a single script. To provide a weak assurance of fairness, haiku-http supports a request timeout: if a handler takes longer than a preconfigured time to process the request (clock time, not CPU time), an HTTP 500 response will be returned to the client: 

{% highlight javascript linenos %}
res.writeHead(200)  
setTimeout(function() {  
   res.end('Hello, world')  
}, 10000)
  

{% endhighlight %}



Note that this is a very inexact method, since a handler that is harmlessly waiting for external stimulus without consuming CPU or memory will also be considered in violation of the policy and terminated.

#### Performing work after completing the HTTP response

Haiku handlers can continue executing after having sent the HTTP response, and there is no mechanism to prevent it from happening. Consider this code:

{% highlight javascript linenos %}
res.writeHead(200)  
res.end('Hello, world')  
doMoreWork()
  

{% endhighlight %}



Or an asynchronous equivalent:

{% highlight javascript linenos %}
res.writeHead(200)  
setTimeout(doMoreWork, 5000)  
res.end('Hello, world')
  

{% endhighlight %}



Since haiku-http cannot currently prevent such situations, it assumes they will happen and addresses them with auto-recycling of child processes. After a child process has handled a configurable number of HTTP requests, the process will recycle itself by first closing its listener to stop accepting new HTTP requests, than waiting for active requests to drain, and finally killing itself. The haiku-http master process will then create a new child process in its place. 

### Going forward

Haiku-http is an attempt to create a sub-process, multi-tenant environment for running HTTP APIs implemented in node.js. 

It currently offers a good level of application data isolation. However, challenges remain with respect to resource consumption control and local denial of service prevention. This is going to be the area of focus and development going forward.   }