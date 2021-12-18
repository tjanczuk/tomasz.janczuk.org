---
post_title: Diagnose node.js apps hosted in IIS with iisnode-debug header
date: 2012-11-06
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: diagnose-node.js-apps-hosted-in-iis-with-iisnode-debug-header
post_date_in_url: true
post_og_image: site
post_excerpt: Tomek on Software - shaken, not stirred
tags:
  - post
---




The [iisnode project](https://github.com/tjanczuk/iisnode) allows hosting node.js applications in IIS on Windows. The recent v0.2.0 release not only added [support for WebSockets](http://tomasz.janczuk.org/2012/11/how-to-use-websockets-with-nodejs-apps.html), but also enhanced the diagnostics capabilities of iisnode. This post describes how you can use the new iisnode-debug HTTP response header to get insights into the state of your node.js application.  

### The iisnode-debug header  

The iisnode v0.2.0 module can now add an iisnode-debug HTTP response header to every node.js HTTP response it sends (including WebSocket handshake response):  

 ![image](http://lh4.ggpht.com/-6LdagTwQdFE/UJnjjmXUS1I/AAAAAAAACw8/Io-VGUbkrSg/image_thumb%25255B12%25255D.png?imgmax=800)  

If you look closely at the value of the iisnode-debug header, you will notice it is a URL with several interesting pieces of information included in the URL fragment. Although the diagnostics information can be gleaned directly from inspecting the value of the URL fragment, a more human friendly visualization is provided when you navigate to that URL in the browser, as shown below (the bit.ly URL resolves to a page hosted on GitHub).   

The iisnode-debug header contains information about:  ![image](http://lh3.ggpht.com/-NIfXLME_WDI/UJnjkisLbLI/AAAAAAAACxM/r9e-y9ydNTU/image_thumb%25255B15%25255D.png?imgmax=800)   

* **Processing time**. Latency of processing this HTTP request (time to first byte of response).  
* **Named pipe connection retry count**. Number of times iisnode had to repeat the attempt to create a named pipe connection to a node.exe process to transmit the request. A number greater than zero may indicate an overloaded server. A number equal to the value of the [maxNamedPipeConnectionRetry](https://github.com/tjanczuk/iisnode/blob/master/src/samples/configuration/iisnode.yml#L31-34) configuration setting for messages that activate a new node.exe process may indicate the application is taking a long time to initialize before it establishes a listener.  
* **HRESULT**. The Win32 error code of processing this HTTP request by iisnode. Any value other than 0 indicates a failure. This helps in diagnosing issues with failed requests.  
* **Server DNS name**. The DNS name of the server to help identify the server handling the request; useful in scaled out scenarios.  
* **w3wp.exe PID.** The PID of the IIS worker process. IIS worker process that hosts the iisnode module and manages one or more node.exe processes. If you notice the PID of the IIS worker process changing over time, it may indicate a fatal failure in iisnode itself which leads to termination of the IIS worker process.  
* **node.exe PID**. This helps identify which server/process was handling the request, which is important in scaled out scenarios. Changing PIDs over the course of multiple requests also indicate process recycling, which may point to an issue with the application itself (e.g. uncaught exceptions that terminate node.exe, only to be re-created by iisnode).  
* **Memory usage**. This is the working set and pagefile memory consumed by the IIS worker process (w3wp.exe) and the node.exe process that handled the request. This data may help detect memory leaks.  
* **Active node.exe processes serving this application**. Number of node.exe processes created by iisnode to handle the node.js application. iisnode can create multiple node.exe processes and load balance incoming requests between them to allow multi-core CPUs to be fully utilized.  
* **Active HTTP requests in this application.** Number of active HTTP requests across all node.exe processes that handle this node.js application.  
* **Active HTTP requests in this node.exe process.** Number of active HTTP requests handled by the node.exe process that processed current request (current request is included in this number).  
* **Total node.js requests processed by w3wp.exe.** The total number of HTTP requests targeting node.js applications handled by this IIS worker process.  
* **Environment.** This includes the version of iisnode on the server, full DNS name of the server, and path to the node.exe.  
  

Getting access to most of the information above was previously possible only with advanced tools and direct access to the server. Thanks to the iisnode-debug header, you can now conveniently access this information from the browser even if your application is running in shared hosting environments.   

### How to enable the iisnode-debug header  

The iisnode-debug header is by default disabled for performance and security reasons. To enable the iisnode-debug header, set the value of the debugHeaderEnabled property in iisnode.yml or web.config to true:  

 ![image](http://lh3.ggpht.com/-OWva0MUzZbY/UJnjlbg48GI/AAAAAAAACxc/-htzI5QkKak/image_thumb%25255B22%25255D.png?imgmax=800)   

Check out the other iisnode.yml or web.config settings iisnode offers in the [configuration sample](https://github.com/tjanczuk/iisnode/tree/master/src/samples/configuration).   

### Other debugging and diagnostics features in iisnode  

  

Besides the new iisnode-debug header, iisnode continues to offer other diagnostics and debugging features for node.js applications hosted in IIS. Check out the following:  

[Debugging node.js applications hosted in IIS](http://tomasz.janczuk.org/2011/11/debug-nodejs-applications-on-windows.html)      
[Using ETW traces](http://tomasz.janczuk.org/2011/09/using-event-tracing-for-windows-to.html)      
[Logging](http://tomasz.janczuk.org/2011/08/hosting-nodejs-applications-in-iis-on.html)  

Enjoy!  }