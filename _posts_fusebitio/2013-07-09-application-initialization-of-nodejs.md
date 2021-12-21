---
tags: ['post']
post_og_image: 'site'
date: '2013-07-09'  
post_title: Application initialization of Node.js apps running in IIS 8 using iisnode
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: application-initialization-of-node.js-apps-running-in-iis-8-using-iisnode
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




Tired of seeing the “first request” to a Node.js app hosted in IIS taking long? This post describes how to configure your Node.js application running in IIS 8 such that the *node.exe* process is always up and ready to serve the HTTP requests. This is in contrast with the default behavior of IIS, where the node.exe process is only created when needed.     

### Message based activation or application initialization  

Message based activation is one of the benefits of running Node.js applications in IIS 8 using the [iisnode](https://github.com/tjanczuk/iisnode) module. With message based activation, the node.exe process running your application will only be started by IIS when a first HTTP request targeting this application arrives on the machine. Under most circumstances this results in reducing the resource consumption on the machine by avoiding creating idle processes for low traffic applications. Message based activation is the default IIS behavior.   

There are situations, however, when you want your Node.js application to be running at all times, ready to serve HTTP requests, regardless of the extra resource consumption involved. For example, you may want to avoid the increased latency an activating HTTP request would incur due to the startup of the node.exe process and any one-time initialization within your Node.js code (e.g. Express.js initialization). You can use the ** [application initialization](http://www.iis.net/learn/get-started/whats-new-in-iis-8/iis-80-application-initialization) feature introduced in IIS 8 to ensure the node.exe process handling your application is always up and running.   

### Enable application initialization for Node.js apps running in IIS/iisnode  

Processing of HTTP requests targeting Node.js applications running in IIS requires two processes: the *w3wp.exe* IIS worker process that runs the [iisnode](https://github.com/tjanczuk/iisnode) module code, and a *node.exe* process running your Node.js application, which iisnode communicates with. Both processes must be started early for application initialization to work.   

To enable application initialization for a Node.js app running in IIS, you must make changes in both *applicationHost.config* and *web.config* files. The *applicationHost.config* is a per-machine IIS configuration file, and changes in it normally require administrative privileges.   

You must make two changes in the *applicationHost.config* file. First, enable *autoStart* and set *startMode* to *AlwaysRunning* on the IIS application pool which runs your Node.js application, e.g:   

{% highlight javascript linenos %}
   <applicationPools>  
  <add name="DefaultAppPool" autoStart="true" startMode="AlwaysRunning" />  
</applicationPools>
  

{% endhighlight %}



Setting these properties will ensure that whenever IIS starts on the machine, the *DefaultAppPool* will also be started, and that an IIS worker process *w3wp.exe* to handle this application pool will be created without waiting for an activating HTTP request. 

In addition to ensuring that the IIS worker process (*w3wp.exe)* is always running, we must also ensure that the actual *node.exe* process process handling your application is started along with it. The *node.exe* process is normally created by the iisnode module running within the IIS worker process only when a first HTTP request targeting that Node.js application arrives. To force iisnode to create a *node.exe* process as soon as *w3wp.exe* itself is started, you must fake an HTTP request that targets your Node.js application using IIS 8’s *preload* feature. The feature simulates a new incoming HTTP request with a specific URL without actually generating any network traffic. To enable preload feature for your Node.js application, you must modify the application entry in *applicationHost.config* with *preloadEnabled* attribute set to *true*, e.g.:

{% highlight javascript linenos %}
<site name="Default Web Site" id="1">  
    <application path="/autostart" preloadEnabled="true" applicationPool="DefaultAppPool">  
        <virtualDirectory path="/" physicalPath="C:\projects\autostart" />  
    </application>  
</site>
  

{% endhighlight %}



The configuration above will cause IIS to generate a fake preload request targeting your Node.js application as soon as the *w3wp.exe* process has started. The URL of the request will by default be the root URL of the application, in the example above */autostart*. You must ensure that the configuration of your IIS application recognizes this request as targeting your Node.js application. The typical way to achieve this is to use the URL rewriting within the *web.config* of your application to rewrite all incoming traffic to the entry point of your Node.js application, for example *server.js*  file, e.g.:

{% highlight javascript linenos %}
<configuration>  
  <system.webServer>  
    <handlers>  
      <add name="iisnode" path="server.js" verb="*" modules="iisnode" />   
    </handlers>      
    <rewrite>  
      <rules>  
        <rule name="DynamicContent">  
          <action type="Rewrite" url="server.js" />   
        </rule>  
      </rules>  
    </rewrite>      
  </system.webServer>  
</configuration>
  

{% endhighlight %}



If, for whatever reason, you don’t want to use URL rewriting in your *web.config*, you can specify an alternative URL for IIS to use when issuing the preload request. This is accomplished with the *applicationInitialization* section of *web.config*: 

{% highlight javascript linenos %}
<configuration>  
  <system.webServer>    
    <applicationInitialization skipManagedModules="true" >  
      <add initializationPage="/server.js" />  
    </applicationInitialization>  
    <handlers>  
      <add name="iisnode" path="server.js" verb="*" modules="iisnode" />   
    </handlers>  
  </system.webServer>  
</configuration>
  

{% endhighlight %}



### Nice and warm

With all the steps above executed correctly, the IIS worker process as well as the Node.js process running your application are started as soon as IIS itself starts up, without the need for an activating HTTP request. 

 ![image](http://lh4.ggpht.com/-AVixvFzYtPw/UdwaA-ykcJI/AAAAAAAADoE/xsWZcwQBzW8/image_thumb%25255B1%25255D.png?imgmax=800) 

With that, you can kiss the activation latency goodbye. To compare, here is the request latency of a first request to a lightweight *Hello, world* Node.js app with message based activation:

 ![image](http://lh4.ggpht.com/-43WjtAwubIw/UdwaFvUFG_I/AAAAAAAADoU/E_nmRfzulKc/image_thumb%25255B4%25255D.png?imgmax=800) 

And here is latency of a first request to the same Node.js app configured to use application initialization:

 ![image](http://lh5.ggpht.com/-2UefLfLEXvQ/UdwaHF8AcdI/AAAAAAAADok/R56Eh5ePVIc/image_thumb%25255B6%25255D.png?imgmax=800) 

Enjoy!  }