---
post_title: Debug Node.js applications in Windows Azure Web Sites
date: 2013-07-29
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: debug-node.js-applications-in-windows-azure-web-sites
post_date_in_url: true
post_og_image: site
post_excerpt: Tomek on Software - shaken, not stirred
tags:
  - post
---




This post explains how you can remotely debug your Node.js application deployed to Windows Azure Web Sites using the node-inspector debugger.   

Node.js applications deployed to Azure are using the [iisnode](https://github.com/tjanczuk/iisnode) module to run. One of the features of iisnode is [integrated debugging experience based on node-inspector](http://tomasz.janczuk.org/2011/11/debug-nodejs-applications-on-windows.html). In order to enable node-inspector debugger in Azure, a few steps need to be followed.   

 ![image](http://lh6.ggpht.com/-60hsCFj4XaE/UfaLd44oOkI/AAAAAAAAD1s/gyLg7t5TT1U/image_thumb%25255B3%25255D.png?imgmax=800)   

### Configuration  

To enable node-inspector debugging for Node.js apps deployed to Azure, you must currently ensure the settings in your *iisnode.yml* and *web.config* files are correct. We are working on streamlining this experience in future releases of Windows Azure Web Sites.   

In *iisnode.yml*, you must enable debugging by setting the *debuggingEnabled* property to true:   

{% highlight yaml linenos %}
   debuggingEnabled: true
  

{% endhighlight %}





In *web.config*, you must configure URL rewriting rules which allow *iisnode* to distinguish HTTP requests that target the node-inspector debugger from requests that target your application. Assuming the entry point to your Node.js application is the *server.js* file, your *web.config* could look as follows:

{% highlight xml linenos %}
<?xml version="1.0" encoding="utf-8"?>  
<configuration>  
  <system.webServer>  
    <handlers>  
      <add name="iisnode" path="server.js" verb="*" modules="iisnode"/>  
    </handlers>  
    <rewrite>  
      <rules>  
        <rule name="NodeInspector" patternSyntax="ECMAScript" stopProcessing="true">  
          <match url="^server.js\/debug[\/]?" />  
        </rule>            
        <rule name="Application">  
          <action type="Rewrite" url="server.js"/>  
        </rule>  
      </rules>  
    </rewrite>  
  </system.webServer>  
</configuration>
  

{% endhighlight %}



### Using the debugger

After your application has been re-deployed with the changes to *web.config* and *iisnode.yml* described above, you are ready to start debugging. 

To open the debugger for your application, navigate to *http://yourapp.azurewebsites.net/server.js/debug*. This should bring up the familiar node-inspector interface for your application, which allows you to set breakpoints, inspect code, etc. In a separate browser window you can invoke an endpoint in your application, e.g. *http://yourapp.windowsazure.net/apis/myapi*. If any of the breakpoints you set in node-inspector were hit, you should see execution paused in the browser instance running node-inspector. 

When you are done with the debugging session or simply want to start from a clean slate, navigate to *http://yourapp.windowsazure.net/server.js/debug?kill*. This will terminate the debugger and debugee processes in Azure. 

When you have finished debugging, remember to disable the feature by setting *debuggingEnabled* to *false* in *iisnode.yml*. The URL rewrite rules in *web.config* can remain in place. 

You can read more about the node-inspector integration with iisnode [here](http://tomasz.janczuk.org/2011/11/debug-nodejs-applications-on-windows.html). 

### Advanced configuration

The iisnode debugger integration requires that part of the URL space of your Windows Azure Web Site is reserved for use by node-inspector. However, you have control over the URL path segment value used for that purpose. By default the value of the segment is *debug*, and so you navigate to the node-inspector debugger by visiting *http://yourapp.azurewebsites.net/server.js/debug*. You can modify this value using the *debuggerPathSegment* setting in *iisnode.yml*, e.g.:

{% highlight yaml linenos %}
debuggerPathSegment: 6534adw287dgx552
  

{% endhighlight %}



When changing the debugger path segment, a corresponding change must be done in the URL rewrite rules in *web.config*. Once the path segment has been changed, you can navigate to the node-inspector debugger using the *http://yourapp.azurewebsites.net/server.js/6534adw287dgx552* URL. 

Note that customizing the URL path segment reserved for debugging is important for the security of your site. As long as debugging is enabled in *iisnode.yml* (it is disabled by default), anyone who knows the value of the debugger path segment can interfere with your application. It is therefore wise to set the value to a cryptographically secure string before enabling debugging for your site. 

Enjoy!  }