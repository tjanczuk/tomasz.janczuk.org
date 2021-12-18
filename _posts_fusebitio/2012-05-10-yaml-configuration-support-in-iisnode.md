---
post_title: YAML configuration support in iisnode
date: 2012-05-10
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: yaml-configuration-support-in-iisnode
post_date_in_url: true
post_og_image: site
post_excerpt: Tomek on Software - shaken, not stirred
tags:
  - post
---




Upon popular demand for YAML configuration (or rather lack of demand for angle brackets of web.config), [iisnode v0.1.20](https://github.com/tjanczuk/iisnode) adds support for specifying configuration options in a YAML file. This allows you to use an established, user-friendly format to control most aspects of [hosting node.js applications in IIS on Windows using iisnode](http://tomasz.janczuk.org/2011/08/hosting-nodejs-applications-in-iis-on.html).   

### iisnode.yml  

You can use iisnode.yml file to control several configuration options for iisnode. For a full and current list with a description see [here](https://github.com/tjanczuk/iisnode/blob/master/src/samples/configuration/iisnode.yml).  

{% highlight yaml linenos %}
   # The optional iisnode.yml file provides overrides of   
# the iisnode configuration settings specified in web.config.  
  
node_env: production  
nodeProcessCommandLine: "c:\program files\nodejs\node.exe"  
interceptor: "c:\program files\iisnode\interceptor.js"  
nodeProcessCountPerApplication: 1  
maxConcurrentRequestsPerProcess: 1024  
maxNamedPipeConnectionRetry: 24  
namedPipeConnectionRetryDelay: 250  
maxNamedPipeConnectionPoolSize: 512  
maxNamedPipePooledConnectionAge: 30000  
asyncCompletionThreadCount: 0  
initialRequestBufferSize: 4096  
maxRequestBufferSize: 65536  
watchedFiles: *.js;iisnode.yml  
uncFileChangesPollingInterval: 5000  
gracefulShutdownTimeout: 60000  
loggingEnabled: true  
logDirectory: iisnode  
debuggingEnabled: true  
debuggerPortRange: 5058-6058  
debuggerPathSegment: debug  
maxLogFileSizeInKB: 128  
maxTotalLogFileSizeInKB: 1024  
maxLogFiles: 20  
devErrorsEnabled: true  
flushResponse: false  
enableXFF: false  
promoteServerVars: 

{% endhighlight %}



The iisnode.yml configuration file is not a replacement of web.config: both can exist at the same time. However, if iisnode.yml is present, its settings override the values specified in the system.webServer\iisnode section of web.config. 

### Autoupdate

By default, when the iisnode.yml configuration file changes, iisnode will gracefully recycle the node.js application. Active requests will be allowed to finish, but all new requests will be handled by a newly created node.js process that uses new configuration values. 

### So why do I still need web.config?

Even if you use iisnode.yml file to configure your node.js application running in iisnode, you still need to include a web.config file. Since IIS is a hosting environment that supports a vast (and extensible) variety of content types running side by side (static files, ASP.NET apps, PHP, node.js , …), web.config is a mechanism to tell IIS that your node.js entry point (e.g. server.js) should be handled by iisnode. In addition, web.config provides the full control over how IIS behaves and can add unique value to your node.js application as explained below. 

The good news is that in majority of situations you can use a boilerplate web.config below and never come back to it, relying solely on iisnode.yml for configuration of node.js applications running in IIS. 

{% highlight xml linenos %}
<?xml version="1.0" encoding="utf-8"?>  
<configuration>  
    <system.webServer>           
      <handlers>  
           <add name="iisnode" path="server.js" verb="*" modules="iisnode"/>  
     </handlers>  
      <rewrite>  
           <rules>  
                <rule name="LogFile" patternSyntax="ECMAScript" stopProcessing="true">  
                     <match url="iisnode"/>  
                </rule>  
                <rule name="NodeInspector" patternSyntax="ECMAScript" stopProcessing="true">                      
                    <match url="^server.js\/debug[\/]?" />  
                </rule>  
                <rule name="StaticContent">  
                     <action type="Rewrite" url="public{{ "{{" }}REQUEST_URI}}"/>  
                </rule>  
                <rule name="DynamicContent">  
                     <conditions>  
                          <add input="{{ "{{" }}REQUEST_FILENAME}}" matchType="IsFile" negate="True"/>  
                     </conditions>  
                     <action type="Rewrite" url="server.js"/>  
                </rule>  
           </rules>  
      </rewrite>  
   </system.webServer>  
 </configuration>

{% endhighlight %}



The web.config above has the following effect:

1. It specifies server.js as the entry point to your node.js application.  
2. It redirects all requests for URLs that map to physical files in the “public” subdirectory to an IIS static file handler. Using IIS static file handler has a large performance benefit compared to serving static content from within a node.js application. The handler leverages IIS and OS low level caching mechanisms which offer superb performance.  
3. It allows IIS to serve the log files that capture output of a node.js application as static files such that you can access them over HTTP. By default, if your server.js entry point is reached at http://mysite.com/server.js, the log files would be accessible at [http://mysite.com/iisnode](http://mysite.com/iisnode).  
4. It exposes the built-in node-inspector debugger at http://mysite.com/server.js/debug. [Learn more about debugging with iisnode](http://tomasz.janczuk.org/2011/11/debug-nodejs-applications-on-windows.html).  
5. It sends all other HTTP requests to be processed by your node.js application in server.js.  


### Try it out

Get your copy if iisnode at [https://github.com/tjanczuk/iisnode](https://github.com/tjanczuk/iisnode) and give YAML config a try!  }