---
post_title: Debug node.js applications on Windows with iisnode integrated debugging
date: 2011-11-04
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: debug-node.js-applications-on-windows-with-iisnode-integrated-debugging
post_date_in_url: true
post_og_image: site
post_excerpt: Tomek on Software - shaken, not stirred
tags:
  - post
---




The [iisnode](https://github.com/tjanczuk/iisnode) project allows hosting node.js applications in IIS on Windows. As of version 0.1.9, iisnode includes fully integrated debugging experience based on the excellent [node-inspector debugger by Danny Coates](https://github.com/dannycoates/node-inspector).   

 ![image](http://lh5.ggpht.com/-vXiXdN6P9Qc/TrRRjVArcUI/AAAAAAAAB2I/Sx5zl_zG_7I/image_thumb%25255B2%25255D.png?imgmax=800)  

These are the benefits of the integrated debugging in iisnode:  

* **No-config deployment on Windows:** host your node.js application in IIS on Windows using iisnode and you are debug-ready. No special configuration, no additional packages required to enable debugging.  
* **Cross-platform debugging:** access the application as well as the debugger for that application from a browser on any platform (Windows, Mac, Linux, Unix). No client side tools required beyond a WebKit-based web browser.  
* **Debug after deployment, even in shared hosting environments:** both the application and the debugger are exposed on a single port number over HTTP, typically 80. Application and debugger do not require separate externally visible ports, and you don’t need access to the machine to start the debugger process.  
* **Traverse firewalls and proxies**: HTTP long polling based protocol (as opposed to websockets) offers the most firewall- and proxy-friendly notification mechanism for snappy debugging experience.  
  

Let’s have a closer look.   

### Getting started  

For this walkthrough, this is what you need:  

1. Windows machine with IIS 7.x installed  
2. Install [node.js v0.5.10](https://github.com/tjanczuk/node/downloads) or greater (important: iisnode debugging does not work with node.js versions < v0.5.10)  
3. Install iisnode for IIS 7.x for [x86](http://go.microsoft.com/?linkid=9784330) or [x64](http://go.microsoft.com/?linkid=9784331), depending on your OS.  
4. Set up iisnode samples by calling “%programfiles%\iisnode\setupsamples.bat” with administrative privileges.  
5. You need a WebKit enabled web browser. Google Chrome or Safari are some of the choices on Windows. On a Mac as you are set up with Safari or Chrome. iisnode debugging does not currently work in IE or Firefox.  
  

You can also leverage the debugging functionality of iisnode when developing in WebMatrix and IIS Express. Install iisnode for WebMatrix from [here](http://go.microsoft.com/?linkid=9784329). Read more about WebMatrix development with iisnode [here](http://tomasz.janczuk.org/2011/08/developing-nodejs-applications-in.html).   

### Basic debugging scenario  

Navigate to the iisnode samples at [http://localhost/node](http://localhost/node) in your WebKit-based browser, then choose one of the samples (say helloworld). Notice the two links pointing to the node.js application itself and the debugger for that application:  

 ![image](http://lh4.ggpht.com/-5NQBBFN2ivs/TrRRj-KYb3I/AAAAAAAAB2Y/F_fTX75vUqk/image_thumb%25255B8%25255D.png?imgmax=800)  

Click on the link for debugging which will open a new browser window or tab and take you to [http://localhost/node/helloworld/app.js/debug](http://localhost/node/helloworld/app.js/debug). That URL serves up the node-inspector UI. Behind the scenes iisnode started both the helloworld node.js application and the node-inspector debugger and connected them:  

 ![image](http://lh3.ggpht.com/-UpttHGUX5mc/TrRRkEPy2xI/AAAAAAAAB2o/H5GZVONUy2s/image_thumb%25255B7%25255D.png?imgmax=800)    

Go ahead and put a breakpoint within the http callback in line 4. Then open a new browser window and navigate to the actual application at [http://localhost/node/hello.js](http://localhost/node/hello.js). You will notice the browser appears to be waiting for the server response:  

 ![image](http://lh4.ggpht.com/-ateqoALoFHU/TrRRkcwGG7I/AAAAAAAAB24/yTSmfDLs5yc/image_thumb%25255B11%25255D.png?imgmax=800)  

This is because your server side application has now stopped on the breakpoint you set:  

 ![image](http://lh3.ggpht.com/-ACZg4I8sQRg/TrRRlDVW3LI/AAAAAAAAB3I/PhH-QHhzOSs/image_thumb%25255B14%25255D.png?imgmax=800)  

From here, you can use the debugger window to inspect variables, the call stack, evaluate expressions in real time, step through the code (even 3rd party modules and node.js code your application is using) etc. For example, you can easily inspect the request headers of the HTTP request just received using the locals window of the console:  

 ![image](http://lh3.ggpht.com/-RIEK24V_cy8/TrRRlW0RonI/AAAAAAAAB3Y/rjC2X0S4zeY/image_thumb%25255B17%25255D.png?imgmax=800)  

When you are done debugging the application, you need to terminate the debugging session by navigating to [http://localhost/node/helloworld/hello.js/debug?kill](http://localhost/node/helloworld/hello.js/debug?kill). This command will cause iisnode to terminate the debugee and the debugger. Subsequent application requests will cause activation of a new application instance started in a regular, non-debug mode (which is more lightweight):   

 ![image](http://lh3.ggpht.com/-UA1SUNkkly0/TrRRl92nODI/AAAAAAAAB3o/vHt09396mMU/image_thumb%25255B20%25255D.png?imgmax=800)    

A debugging pattern I found convenient is to have 3 browser windows opened at the same time: one for the application, one for debugger UI, and one for the ?kill debugging command. It is then easy to switch between the three windows to initiate application requests, manipulate the debugger, or terminate the debugging session (perhaps to start fresh) by simply refreshing the respective browser windows:  

 ![image](http://lh6.ggpht.com/-I1K8__TtFUE/TrRRmXDwwqI/AAAAAAAAB34/J8_V_1uufYs/image_thumb%25255B24%25255D.png?imgmax=800)  

Note that refreshing the window with the [http://localhost/node/helloworld/hello.js/?kill](http://localhost/node/helloworld/hello.js/?kill) URL requires that you first navigate away from the node-inspector UI in the other window, for reasons explained in the next section.     

### Understanding the debugger interactions  

To effectively use the iisnode integrated debugging, it is important to understand that the debugger maintains server side state and how that state is managed. The state machine below shows server side state transitions in response to different HTTP requests iisnode receives; the pragmatic essence of this is the following:  

* **Closing and reopening the browser does not affect server side state.** In particular server does not attempt to detect and react to the client closing the browser window. That means you can start debugging in one window, close it, open another browser window, navigate to /app.js/debug, and continue where you left off – all breakpoints are preserved. You can also refresh your browser while debugging – you should end up in the same place you left.  
* **Issuing /app.js/debug?kill request is a great way to get started with a clean slate** – the server kills the app.js application regardless if it is running in the debug or regular mode, as well as the debugger (if running) and puts you in the initial state. One word of caution: when issuing /app.js/debug?kill request, make sure no other browser window is opened with the debugger UI – the debugger is continuously issuing HTTP long polling requests which would cause the server to transition back to the debugging state right after the debugger has been killed by the /app.js/debug?kill request.  
* **Refreshing a browser pointing at /app.js/debug[?brk] does not recycle the application if it is already running in debug mode**. This means no debugging state of an already debugged application is lost. In particular, all breakpoints are preserved.  
* **Issuing /app.js/debug[?brk] request will kill the application if it is running in the regular mode, and restart it in debug mode**. If the application is already running in debug mode when this request is received, it is not terminated and restarted, but simply attached to.  
* **Using the three browser windows shown above is a great way to fully control the server side state** by simply refreshing the browser windows.  
  

 ![image](http://lh5.ggpht.com/-IGpKQ4bikXE/TrRRmuOCwGI/AAAAAAAAB4I/Axvqjmp9wHI/image_thumb%25255B27%25255D.png?imgmax=800)  

### Fine tuning  

While the basic debugging scenario in iisnode works out of the box, there are a few options one can configure in the web.config file of the web application to fine tune the experience. They are controlled with the following configuration options, shown below with their default values:   

{% highlight xml linenos %}
   <configuration>  
  <system.webServer>  
    <iisnode        
      debuggerPortRange="5058-6058"  
      debuggerPathSegment="debug"  
      maxNamedPipeConnectionRetry="3"  
      namedPipeConnectionRetryDelay="2000"        
     />  
  </system.webServer>  
</configuration>
  

{% endhighlight %}



The **debuggerPortRange** is the range of TCP ports iisnode will use to set up communication between the node-inspector debugger and the debugee. iisnode only picks up those ports from this range that are not in use. iisnode uses round-robin logic to assign TCP ports from this range for consecutive debugging sessions. 

Integrated debugging in iisnode requires a part of the URL space of the application to be dedicated to interactions with the debugger. The **debuggerPathSegment** provides control over the URL segment following the node.js application name in the HTTP request URL that the iisnode debugger will take over. For example, given a node.js application available at http://foo.com/bar/baz.js, the iisnode debugger will intercept and process all HTTP requests sent to http://foo.com/bar/baz.js/{debuggerPathSegment} (by default http://foo.com/bar/baz.js/debug) and all subordinate URLs. Note that the application itself will typically be reachable using URLs that do not contain the *.js file name in them, since in most situations one will want [to leverage the IIS URL rewrite module to customize the URL space of node.js the application](http://tomasz.janczuk.org/2011/08/using-url-rewriting-with-nodejs.html). 

The **debuggerPathSegment** setting can also be used to obfuscate access to the debugger as a low key approach to securing your service. When the debuggerPathSegment is set to a cryptographically random value (or even a GUID), only folks with knowledge of that string can access the debugger. 

The iisnode debugger re-uses the **maxNamedPipeConnectionRetry** and **namedPipeConnectionRetryDelay** settings [used in non-debug scenarios](https://github.com/tjanczuk/iisnode/blob/master/src/samples/configuration/web.config) to control the initialization of the debugger and debugee. The debugee process is expected to start listening on the debugging TCP port shortly after startup. iisnode will ensure the debugger is only started after the debugee is ready to accept TCP connections on the debugging port. iisnode does it by starting the debugee process, trying to connect to the TCP debugging port itself, and launching the debugger only after a successful connect attempt. The maxNamedPipeConnectionRetry controls the number of connection attempts to the TCP debugging port of the debugee that iisnode will perform, and the namedPipeConnectionRetryDelay controls the time in milliseconds between two consecutive attempts. 

### Under the hood

The integrated debugging in iisnode is based on the [node-inspector debugger by Danny Coates](https://github.com/dannycoates/node-inspector). This is a high level picture of how a typical non-iisnode node-inspector setup works that will help explain the changes that were necessary to integrate it into iisnode:

 ![image](http://lh3.ggpht.com/-8eOtaLJrO48/TrRRnItiUbI/AAAAAAAAB4Y/2ZyJYOqXBdc/image_thumb%25255B33%25255D.png?imgmax=800)

In this standard configuration, [which Glenn Block showed how you can set up on Windows](http://codebetter.com/glennblock/2011/10/13/using-node-inspector-to-debug-node-js-applications-including-on-windows-and-using-ryppi-for-modules/), node-inspector and the debugged application are running as two separate processes on the same machine. They communicate with each other using the [V8 debugging protocol](http://code.google.com/p/v8/wiki/DebuggerProtocol) over a local TCP connection. The debugged application would typically expose an HTTP endpoint over a specific port Z that browser clients can connect to. The node-inspector instance will expose two endpoints: one HTTP endpoint over port X (different than Z) for the HTML interactions with the browser over which node-inspector UI and other static content is obtained, and a websocket endpoints over port Y that is used to relay the V8 debugging protocol communication to the browser. node-inspector uses socket.io to abstract away that duplex communication channel. In this configuration, node-inspector application itself is responsible for serving static content (HTML files, JPG images, client side JavaScript) to the browser, which it does using the paperboy module. 

The integrated debugging in iisnode refactors the system in several ways shown on the picture below:

 ![image](http://lh6.ggpht.com/-MUIjg29IZHc/TrRRnTa0iNI/AAAAAAAAB4o/-Ka_Qq4YC_I/image_thumb%25255B36%25255D.png?imgmax=800)

The key change is that both the browser instance running the debugger and the one running the application communicate with the backend using HTTP and HTTP long polling over a single port number. Websocket communication has been replaced with HTTP long polling. One port number and vanilla HTTP enable more robust deployments to a broader variety of hosting environments, and make it more resilient to a broader variety of proxies between the client and the server than is otherwise possible with websockets. 

When iisnode receives a new HTTP requests, it routes the request to one of three destinations. HTTP long polling requests that relay the V8 debugging protocol are routed to the node-inspector. The requests for static content of the node-inspector UI (HTML, images, client side JavaScript) are re-routed back to IIS which uses the efficient, native static file handler to serve them back to the client (including server side output caching and compression). Finally, application requests are routed to the debugged application. Communication between iisnode and either the debugger or the debugee (both of which are node.js applications) uses HTTP over named pipes for increased efficiency compared to HTTP over TCP. In case of the debugger it is actually HTTP long polling over named pipes. 

### Closing words

With iisnode you can now easily debug node.js applications deployed to IIS from a variety of client platforms, including non-Windows. The design has been optimized to allow debugging after deployment to a broad variety of Windows-based shared hosting environments. The choice of protocols was optimized for robustness when communicating across firewalls and proxies. 

This new feature has just shipped and I do expect issues will be found. I would appreciate your help in making iisnode better: please file any issues you encounter at [https://github.com/tjanczuk/iisnode/issues](https://github.com/tjanczuk/iisnode/issues). 

Enjoy!  }