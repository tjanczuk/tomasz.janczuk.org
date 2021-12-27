---
tags: ['post']
post_og_image: 'site'
date: '2013-01-04'  
post_title: Hosting socket.io WebSocket apps in IIS using iisnode
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: hosting-socketio-websocket-apps-in-iis
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




In this post I explain how to configure a socket.io node.js application to use of WebSockets when hosting it in IIS 8 using [iisnode](https://github.com/tjanczuk/iisnode). This complements a recent post in which I showed [how to host node.js WebSocket applications in IIS on Windows using iisnode and faye-websocket module](http://tomasz.janczuk.org/2012/11/how-to-use-websockets-with-nodejs-apps.html).   

The complete code of the sample for self-hosting and IIS hosting of socket.io and faye-websocket in IIS using iisnode is available at [https://github.com/tjanczuk/dante](https://github.com/tjanczuk/dante).   

### From Zero to Divine in  Seven Seconds  

You need Windows 8 or Windows 2012 machine with IIS 8 and [iisnode](https://github.com/tjanczuk/iisnode) installed.   

Clone the Dante sample, install dependencies, and set up IIS virtual directory pointing to the code of the sample:  

```

   git clone https://github.com/tanczuk/dante.git  
npm install  
dante\setup.bat
  

```


Then navigate to the socket.io sample at [http://localhost/dante/server-socketio.js](http://localhost/dante/server-socketio.js). You should see Dante’s Divine Comedy streamed to you over websockets from a socket.io application hosted in IIS 8 using iisnode, one stanza every 2 seconds:

 ![image](http://lh4.ggpht.com/-FD1loGRhzvU/UOe-YcJTpxI/AAAAAAAADWw/sICXDWxfTU0/image_thumb%25255B8%25255D.png?imgmax=800) 

You can see four HTTP requests being made. The first one targets the node.js application server-socketio.js and returns the HTML page with client side JavaScript. The page requests the socket.io.js client library from the server, which is the second HTTP call. Next, the client side JavaScript on the page performs the socket.io handshake (3rd HTTP requests). Finally, a WebSocket connection is established and the Dante’s Divine Commedy is streamed from the server as discrete WebSocket messages over that single connection. 





### Under the hood

Hosting socket.io node.js apps in IIS using iisnode requires some extra steps compared to self-hosting such apps. Unlike in a typical self-hosting case, node.js apps hosted in a IIS virtual directory own only a subset of the URL space, and socket.io must be adequately configured to that effect. In addition, IIS must be told which requests constitute socket.io traffic and must be handled by iisnode as opposed to other built-in IIS handlers (e.g. static file handlers). 

This explanation assumes the node.js application is hosted in a IIS virtual directory named ‘dante’ as opposed to the root of an IIS website. The latter case is simpler to configure, with only required changes being the changes in web.config described below.

Below are the key components of the configuration. Full source code of the sample is at [https://github.com/tjanczuk/dante](https://github.com/tjanczuk/dante). 

#### Web.config

There are three aspects that must be configured in web.config: handler registration, URL rewrite rules, and IIS wesocket module configuraiton. 

First, you must inform IIS that the server-socket.io.js file is a node.js application and must be handled by iisnode. Without this, IIS would try to serve the file as a client side JavaScript using the static file handler: 

```
<handlers>  
   <add name="iisnode-socketio" path="server-socketio.js" verb="*" modules="iisnode" />  
</handlers>
  

```


Then, the URL rewrite module must be informed that all HTTP requests that start with the ‘socket.io’ segment constitute node.js traffic and should be redirected to the server-socketio.js as the entry point of the node.js application. Without this, IIS would attempt to map these requests to other handlers, and most likely respond with a failure code:

```
<rewrite>  
     <rules>  
          <rule name="LogFile" patternSyntax="ECMAScript">  
               <match url="socket.io"/>  
               <action type="Rewrite" url="server-socketio.js"/>  
          </rule>  
     </rules>  
</rewrite> 
  

```


Lastly, the built-in WebSocket module that IIS 8 ships with must be turned off, since otherwise it would conflict with the WebSocket implementation provided by socket.io on top of the raw HTTP upgrade mechanism node.js and iisnode support:

```
<webSocket enabled="false" />
  

```


The complete web.config is at [https://github.com/tjanczuk/dante/blob/master/web.config](https://github.com/tjanczuk/dante/blob/master/web.config).

#### The server

The server code must configure socket.io to inform it that the node.js application owns just a subset of the URL space as a result of being hosted in IIS virtual directory. This means that socket.io traffic that the server normally listens to on the /socket.io path is going to arrive at /dante/socket.io:

```
io.configure(function() {  
    io.set('transports', [ 'websocket' ]);  
    if (process.env.IISNODE_VERSION) {  
        io.set('resource', '/dante/socket.io');  
    }  
});
  

```


Notice this configuration change in socket.io is only made when the application is hosted in IIS; the same code base of the sample can also be self-hosted, in which case the configuration is left unmodified. 

The full code of the server is at [https://github.com/tjanczuk/dante/blob/master/server-socketio.js](https://github.com/tjanczuk/dante/blob/master/server-socketio.js).

#### The client

The client code must contain configuration change corresponding to the server, otherwise socket.io client library would by default assume the socket.io traffic should be sent to the /socket.io path on the server:

```
var address = window.location.protocol + '//' + window.location.host;  
var details = {  
    resource: (window.location.pathname.split('/').slice(0, -1).join('/') + '/socket.io').substring(1)  
};  
  
var client = io.connect(address, details); 
  

```


Notice that this client code works correctly both when the server is self-hosted or hosted in a IIS virtual directory. This is because the socket.io configuration sets the URL paths relative to the pathname of the original request that rendered the HTML page. In the self-hosted case, the original page is rendered from [http://localhost:8888/](http://localhost:8888/), and consequently socket.io’s resource property is set to ‘socket.io’. In the IIS hosted case, the original request is rendered from [http://localhost/dante/server-socketio.js](http://localhost/dante/server-socketio.js), and as a result the socket.io resource property will be set to ‘dante/socket.io’. This allows the client code to be agnostic to how the server is hosted. 

The full code of the client is at [https://github.com/tjanczuk/dante/blob/master/index-socketio.html](https://github.com/tjanczuk/dante/blob/master/index-socketio.html).

Enjoy!  