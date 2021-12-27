---
tags: ['post']
post_og_image: 'site'
date: '2011-08-29'  
post_title: Hosting express node.js applications in IIS using iisnode
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: hosting-express-nodejs-applications-in
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




In my last two posts I introduced the [iisnode](https://github.com/tjanczuk/iisnode) project which allows [hosting node.js applications in IIS on Windows](http://tomasz.janczuk.org/2011/08/hosting-nodejs-applications-in-iis-on.html), as well as shown [how it integrates with the URL rewrite module](http://tomasz.janczuk.org/2011/08/using-url-rewriting-with-nodejs.html). In this post I demonstrate how to run node.js applications that use the popular [express](http://expressjs.com/) framework in IIS.   

### Installing express on Windows  

Node.js modules, including express, are typically installed using [NPM](http://npmjs.org/). The bad news is that as of this writing NPM is not yet supported on Windows. The good news is that for simple cases one can use the [ryppi.py](https://github.com/japj/ryppi) script. Assuming you have [Python installed](http://www.activestate.com/activepython/downloads), you can call:  

```

   ryppi.py install express

```


which will create the node_modules folder with the downloaded express library. You can check out the resulting layout [here](https://github.com/tjanczuk/iisnode/tree/master/src/samples/express). 

### The code

A simple express application we will host in IIS looks like this:

```
 var express = require('express');

 var app = express.createServer();

 app.get('/node/express/hello/foo', function (req, res) {
     res.send('Hello from foo! [express sample]');
 });

 app.get('/node/express/hello/bar', function (req, res) {
     res.send('Hello from bar! [express sample]');
 });

 app.listen(process.env.PORT);

```


Two key aspects to call out that may be different from your bread & butter express app are:

1. The path specified in app.get calls must be the full path of the request (lines 5 and 9). When your app is hosted in IIS, depending on the configuration you may not necessarily own the entire namespace over port 80. Like in the example above, your IIS hosted express application may reside in the ‘express’ folder of  the ‘node’ virtual directory, and only own the subordinate URL namespace.  
2. Similarly to a non-express node.js app hosted in iisnode, the listen port is provided by IIS through the PORT environment variable. When you start your listener (line 13), this is the port you should specify.  


### The web.config

I talked about using the URL rewrite module for regular node.js applications before, and URL rewriting is perhaps even more relevant in case of URL-conscious express apps. The web.config below allows the express application saved in hello.js to receive HTTP requests directed at all URL paths subordinate to the ‘hello’ path component, as configured in lines 20-27:

```
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
     
         http://localhost/node/express/hello/foo
         http://localhost/node/express/hello/bar
         
     -->

     <rewrite>
       <rules>
         <rule name="hello">
           <match url="hello/*" />
           <action type="Rewrite" url="hello.js" />
         </rule>
       </rules>
     </rewrite>

     <!-- exclude node_modules directory and subdirectories from serving
     by IIS since these are implementation details of node.js applications -->
     
     <security>
       <requestFiltering>
         <hiddenSegments>
           <add segment="node_modules" />
         </hiddenSegments>
       </requestFiltering>
     </security>    
     
   </system.webServer>
 </configuration>

```


One other aspect worth pointing out is request filtering. Remember the express application relies on the express library installed in the node_modules directory? You probably don’t want the contents of this directory to be served by IIS in any shape or form, and you can express (sic!) that desire by adding it to hidden segments list (lines 32-38). 

### Voila!

Your IIS-hosted express node.js application behaves like expected:

 ![image](http://lh3.ggpht.com/-jzVceDfgFjs/TlxGqI8qGoI/AAAAAAAAB0M/EsbluCYy1sA/image_thumb%25255B2%25255D.png?imgmax=800)

 ![image](http://lh4.ggpht.com/-ou6wfv14eE0/TlxGq22AztI/AAAAAAAAB0U/zOM5nRo81ro/image_thumb%25255B3%25255D.png?imgmax=800)

### So where can I get iisnode again?

Everything you need to get started is at [https://github.com/tjanczuk/iisnode](https://github.com/tjanczuk/iisnode). Make sure to check out the [express sample](https://github.com/tjanczuk/iisnode/tree/master/src/samples/express). Feedback welcome!  