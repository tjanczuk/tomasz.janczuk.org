---
tags: ['post']
post_og_image: 'site'
date: '2012-11-01'  
post_title: How to use WebSockets with node.js apps hosted in iisnode
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: how-to-use-websockets-with-node.js-apps-hosted-in-iisnode
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




*Note: if you are interested in hosting **socket.io** WebSocket applications in IIS using iisnode, check out the more recent post at [http://tomasz.janczuk.org/2013/01/hosting-socketio-websocket-apps-in-iis.html](http://tomasz.janczuk.org/2013/01/hosting-socketio-websocket-apps-in-iis.html)*  

The recent release of [iisnode v0.2.0](https://github.com/tjanczuk/iisnode) enables using WebSockets in node.js applications hosted in IIS 8.  This functionality requires Windows Server 2012 or Windows 8. To read more about iisnode in general check out [my previous post](http://tomasz.janczuk.org/2011/08/hosting-nodejs-applications-in-iis-on.html).   

### Getting started  

After installing iisnode v0.2.0, get a sample WebSocket application from [https://github.com/tjanczuk/dante](https://github.com/tjanczuk/dante) (if you are not a Git person, you can also [download a ZIP](https://github.com/tjanczuk/dante/zipball/master)):   

```

   git clone https://github.com/tjanczuk/dante.git
  

```


Next, download the node.js dependencies and set up an IIS application in IIS 8 that points to the downloaded code:

```

npm install  
setup.bat

```


Lastly, navigate to [http://localhost/dante/server-faye.js](http://localhost/dante/server-faye.js) and enjoy Dante’s Dive Comedy, Canto 1, streamed to you over WebSockets though IIS and iisnode, once stanza every 2 seconds:

 ![image](http://lh6.ggpht.com/-Eggt3XQFEHI/UJKyLnttQ-I/AAAAAAAACwg/js7GTxoD4iU/image_thumb%25255B1%25255D.png?imgmax=800)

### Under the hood

If you have a closer look at server.js, you will notice the application uses the [faye-websocket](https://github.com/faye/faye-websocket-node) module to establish a WebSocket server, just like a self-hosted node.js WebSocket application would:

```
var WebSocket = require('faye-websocket')  
    , http = require('http');  
  
var server = http.createServer(handler);  
  
server.addListener('upgrade', function(request, socket, head) {  
    var ws = new WebSocket(request, socket, head);  
    // ...  
});  
  
function handler (req, res) {  
    // ...  
}  
  
server.listen(process.env.PORT || 8888);  

  

```


In fact, you could take this application and run it self-hosted using node.exe without any changes at all. 

The iisnode module uses the functionality enabled in IIS 8 on Windows Server 2012 and Windows 8 to expose the HTTP Upgrade mechanism to the node.js application. Modules like [faye-websocket](https://github.com/faye/faye-websocket-node), [ws](https://github.com/einaros/ws), or [socket.io](http://socket.io/) implement the WebSocket protocol on top of the HTTP Upgrade mechanism and expose WebSocket functionality to the application. 

Using WebSockets in a node.js applications running in iisnode requires that – contrary to what one would expect – websockets are *disabled* in web.config:

```
<configuration>  
  <system.webServer>  
    <webSocket enabled="false" />  
    <handlers>  
      <add name="iisnode" path="server-faye.js" verb="*" modules="iisnode" />  
    </handlers>  
  </system.webServer>  
</configuration>
  

```


This is required because IIS 8 provides its own implementation of the WebSocket protocol that builds on top of the HTTP Upgrade mechanism. If the IIS 8 WebSocket module remained enabled, it would conflict with the WebSocket implementation provided by the node.js application itself in the form of one of the node.js modules, e.g. faye-websocket, ws, or socket.io. The IIS 8 WebSocket module is used to enable WebSocket functionality in ASP.NET applications. 

### So where does it leave you?







The release of iisnode 0.2.0 closes the last major functional gap between self-hosting and IIS hosting node.js applications on Windows: the availability of WebSockets. You can now host your socket.io application in IIS 8 using WebSockets as opposed to falling back to HTTP long polling. Note that WebSocket functionality is only supported on IIS 8 running on Windows Server 2012 or Windows 8. WebSocket on!  }