---
tags: ['post']
post_og_image: 'site'
date: '2013-09-06'  
post_title: Access Windows Azure Cache Service from Node.js and Express
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: access-windows-azure-cache-service-from-node.js-and-express
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




The [Windows Azure Cache Service](http://weblogs.asp.net/scottgu/archive/2013/09/03/windows-azure-new-distributed-dedicated-high-performance-cache-service-more-cool-improvements.aspx) is a great mechanism for scaling out web applications deployed to Windows Azure Web Sites. It allows you to externalize and very quickly access session state from any instance of your web application.  

With the [azurecache](https://github.com/tjanczuk/azurecache) module you can now access Windows Azure Cache Service from Node.js applications. The *azurecache* module allows you to connect to the cache service directly, but it also provides an implementation of a session store that can be used in any session-enabled Express application.   

The *azurecache* module uses [Edge.js](http://tjanczuk.github.io/edge/) to call into the .NET Windows Azure Cache Service client that ships as a NuGet package. As such the module only works on Windows.    

### Using Windows Azure Cache Service to store Express session state  

First create your Windows Azure Cache Service instance following instructions at [Scott Guthrie's blog](http://weblogs.asp.net/scottgu/archive/2013/09/03/windows-azure-new-distributed-dedicated-high-performance-cache-service-more-cool-improvements.aspx). You will end up with an *endpoint URL* of your cache service (e.g. *tjanczuk.cache.windows.net*) and an *access key* (a long Base64 encoded string).  

Then install the *azurecache* and *express* modules:  

```
   npm install azurecache  
npm install express  

  

```


Next author your Express application that uses the *azurecache* module to store Express session state in the Windows Azure Cache Service:

```
var express = require('express')  
    , AzureCacheStore = require('azurecache')(express);  
  
var app = express();  
  
app.use(express.cookieParser());  
app.use(express.session({ store: new AzureCacheStore(), secret: 'abc!123' }));  
  
app.get('/inc', function (req, res) {  
    req.session.counter = (req.session.counter + 1) || 1;  
    res.send(200, 'Increased sum: ' + req.session.counter);  
});  
  
app.get('/get', function (req, res) {  
    res.send(200, 'Current sum: ' + req.session.counter);  
});  
  
app.listen(process.env.PORT || 3000);  

  

```


Lastly set some environment variables and start your server:

```
set AZURE_CACHE_IDENTIFIER={your_azure_cache_endpoint_url}  
set AZURE_CACHE_TOKEN={your_azure_cache_access_key}  
node server.js  

  

```


Every time you visit *http://localhost:3000/inc* in the browser you will receive an ever increasing counter value. When you visit *http://localhost:3000/get* you will receive the current counter value. The value of the counter is stored as part of the Express session state in the Windows Azure Cache Service with a default TTL of one day. You can now scale out the application to several instances since the session state is externalized to the Windows Azure Cache Service. 

### Deploying Node.js apps using Azure Cache Service to Azure Web Sites

If you are not familiar with deploying Node.js application to Windows Azure Web Sites, [this walkthrough](http://www.windowsazure.com/en-us/develop/nodejs/tutorials/create-a-website-(mac)/) will explain the process. 

Deploying an Express application that uses the *azurecache* module to store session state requires that module dependencies are declared in the *package.json* file:

```
{  
  "name": "azurecachetest",  
  "version": "0.1.0",  
  "dependencies": {  
    "express": "3.3.8",  
    "azurecache": "0.1.0"  
  }  
}
  

```


Once you deploy a Node.js application consisting of the *package.json* and *server.js* above to Windows Azure Web Sites, you still need to provide the credentials to Windows Azure Cache Service to it. Just as you were doing this using environment variables before, you can now set the application settings of your web site using the Windows Azure management portal: 

 ![image](http://lh6.ggpht.com/-iN14ezMk41s/Uion2HX3aLI/AAAAAAAAD2E/GroOrOV5ZXU/image_thumb%25255B1%25255D.png?imgmax=800) 

You can also use the management portal to scale out your Express application to multiple instances, now that the session state is externalized to Windows Azure Cache Service:

 ![image](http://lh6.ggpht.com/-zc0tRtqK6VU/Uion2xEgAuI/AAAAAAAAD2U/ZFKRtth1kjY/image_thumb%25255B3%25255D.png?imgmax=800) 





After saving the changes, you can navigate to your site and see the *azurecache* module in action:

 ![image](http://lh5.ggpht.com/-8rcFMLFagHY/Uion4sYkXKI/AAAAAAAAD2k/pF1D9iQ05Xc/image_thumb%25255B5%25255D.png?imgmax=800) 

### How fast is the cache?

What is the latency of accessing Windows Azure Cache Service from a Node.js application using the *azurecache* module? To find out, let’s deploy a simple latency test to Azure Web Sites. The HTTP server will execute 1000 sequential *puts* against the cache and return the average latency in milliseconds as an HTTP response:

```
var http = require('http')  
    , cache = require('azurecache').create();  
  
http.createServer(function (req, res) {  
    var start = Date.now();  
    var count = 1000;  
    function one() {  
        cache.put('puttest', { first: 'Tomasz', last: 'Janczuk' }, function (error) {  
            if (error) throw error;  
            if (--count === 0) {  
                res.writeHead(200, { 'Content-Type': 'text/plain' });  
                res.end('' + ((Date.now() - start) / 1000));  
            }  
            else {  
                one();  
            }  
        })  
    }  
    one();  
}).listen(process.env.PORT || 3000);
  

```




Save, deploy to Azure Web Sites, and send a request:

 ![image](http://lh4.ggpht.com/-um8pkBqq2u0/Uion6r9PyaI/AAAAAAAAD20/YZrOeF4pCF4/image_thumb%25255B7%25255D.png?imgmax=800) 

The average latency of inserting into the cache is just a notch over 1 millisecond. Converting the test to measure the latency of *getting* data from the cache is trivial. Here is the result:

 ![image](http://lh3.ggpht.com/-S7nCXAS3PkM/Uion9HkERFI/AAAAAAAAD3E/MJ_5IW-pjNY/image_thumb%25255B9%25255D.png?imgmax=800) 

Similarly to *put*, a *get* is around 1 millisecond. 

Note that you can only achieve such low latency for Node.js applications deployed to Windows Azure, since locality of data is a major factor in caching. If you run the same performance test by hosting the Node.js server on your developer machine, your latencies will be much higher (in my case they were around 50ms) since every call to the Windows Azure Cache Service needs to go from your developer machine to a Windows Azure data center. 

So, go forth and scale out!   }