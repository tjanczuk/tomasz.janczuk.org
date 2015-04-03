---
layout: post
title: Using URL rewriting with node.js applications hosted in IIS using iisnode
date: '2011-08-29T18:42:00.001-07:00'
author: Tomasz Janczuk
tags:
- JavaScript
- HTTP
- node.js
- IIS
- iisnode
- Windows
- web
modified_time: '2011-08-31T10:33:43.051-07:00'
thumbnail: http://lh6.ggpht.com/-pOwktyycuIo/Tlw_-98O_lI/AAAAAAAABz8/Ree6RYZo1vE/s72-c/image_thumb%25255B2%25255D.png?imgmax=800
blogger_id: tag:blogger.com,1999:blog-2987032012497124857.post-785315689232792194
blogger_orig_url: http://tomasz.janczuk.org/2011/08/using-url-rewriting-with-nodejs.html
---




In [my last post](http://tomasz.janczuk.org/2011/08/hosting-nodejs-applications-in-iis-on.html) I introduced the [iisnode](https://github.com/tjanczuk/iisnode) project which allows hosting node.js applications in IIS on Windows. In this article I discuss using URL rewriting with node.js apps hosted in IIS, functionality necessary in all but the most trivial IIS hosted node.js applications.    

### The problem  

Consider the [hello world](https://github.com/tjanczuk/iisnode/blob/master/src/samples/helloworld/) sample code, saved in the hello.js file in IIS virtual directory:  

{% highlight javascript linenos %}
   var http = require('http');  
  
http.createServer(function (req, res) {  
    res.writeHead(200, {'Content-Type': 'text/plain'});  
    res.end('Hello, world! [helloworld sample]');  
}).listen(process.env.PORT);  
  

{% endhighlight %}



along with the following web.config that registers the iisnode module as a handler of the hello.js file, therefore indicating it is a node.js application:

{% highlight xml linenos %}
<configuration>  
  <system.webServer>  
    <handlers>  
      <add name="iisnode" path="hello.js" verb="*" modules="iisnode" />  
    </handlers>      
  </system.webServer>  
</configuration>
  

{% endhighlight %}



When both hello.js and web.config above are saved in the ‘node’ virtual directory in IIS, one can navigate to the node.js application using the following URL:

{% highlight text linenos %}
http://localhost/node/hello.js
  

{% endhighlight %}



As expected, IIS will realize the hello.js file maps to the iisnode handler and invoke it, and as expected a few million CPU cycles later a ‘Hello, world’ is sent back to the client. 

However, when a subordinate URL path is requested, IIS returns an error, e.g: 

 ![image](http://lh6.ggpht.com/-pOwktyycuIo/Tlw_-98O_lI/AAAAAAAABz8/Ree6RYZo1vE/image_thumb%25255B2%25255D.png?imgmax=800)

The reason for this is that IIS does not understand that paths subordinate to the ‘hello.js’ component of the path should all be handled by the hello.js application. This is unacceptable for all but the most simplistic node.js applications, which typically own the entire URL space. Fortunately, it is easily remedied with the URL rewriting module.

### URL rewriting module to the rescue

Fixing this problem requires configuring the URL rewriting module to indicate that the section of the request path that is subordinate to the hello.js component should be handled by the handler for the hello.js component itself. From IIS perspective the request processing occurs as if the request was made for http://localhost/node/helloworld/hello.js, but the original URL path is preserved and available for the handler to act on. URL rewriting is specified in the web.config file as follows (lines 17-24):

{% highlight xml linenos %}
 <configuration>
   <system.webServer>
     <!-- indicates that the hello.js file is a node.js application 
     to be handled by the iisnode module -->
     <handlers>
       <add name="iisnode" path="hello.js" verb="*" modules="iisnode" />
     </handlers>
     <!-- use URL rewriting to redirect the entire branch of the URL namespace
     to hello.js node.js application; for example, the following URLs will 
     all be handled by hello.js:
     
         http://localhost/node/urlrewrite/hello
         http://localhost/node/urlrewrite/hello/foo
         http://localhost/node/urlrewrite/hello/foo/bar/baz?param=bat
         
     -->    
     <rewrite>
       <rules>
         <rule name="hello">
           <match url="hello/*" />
           <action type="Rewrite" url="hello.js" />
         </rule>
       </rules>
     </rewrite>
   </system.webServer>
 </configuration>

{% endhighlight %}



The configuration above will not only cause all URLs subordinate to hello.js to handled by hello.js node application; it also allows the hello.js file name to be completely removed from the URL path. With the configuration above the node.js service can now be accessed using any of the following URLs:

{% highlight text linenos %}
http://localhost/node/hello/foo/bar/baz  
http://localhost/node/hello/1/2/3  
http://localhost/node/hello/f/b/b?param=bat  
...
  

{% endhighlight %}



### What request URL does the node.js application see?

The request URL passed by IIS to the node.js application is the original URL from before re-writing. For example:

 ![image](http://lh5.ggpht.com/-bcR21m4NZ9k/Tlw__gf6kjI/AAAAAAAAB0E/56bs8sPn7M8/image_thumb%25255B4%25255D.png?imgmax=800)

### Where do I get iisnode again?

Everything you need to get started is at [https://github.com/tjanczuk/iisnode](https://github.com/tjanczuk/iisnode). Make sure to check out the [urlrewrite sample](https://github.com/tjanczuk/iisnode/tree/master/src/samples/urlrewrite). Feedback welcome!  