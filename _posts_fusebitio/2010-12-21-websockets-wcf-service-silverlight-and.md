---
tags: ['post']
post_og_image: 'site'
date: '2010-12-21'  
post_title: Introducing the WebSockets prototype
  draft-montenegro-hybi-upgrade-hello-handshake-00
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: introducing-the-websockets-prototype-draft-montenegro-hybi-upgrade-hello-handshake-00
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




In [my blog post from last summer](http://tomasz.janczuk.org/2010/07/silverlight-html5-websocket-client-with.html) I wrote about a prototype .NET implementation of two drafts of the WebSockets protocol specification - [draft-hixie-thewebsocketprotocol-75](http://tools.ietf.org/html/draft-hixie-thewebsocketprotocol-75) and  [draft-hixie-thewebsocketprotocol-76](http://tools.ietf.org/html/draft-hixie-thewebsocketprotocol-76) - making its way through the IETF at that time. Since then, there [have been a number](http://tools.ietf.org/wg/hybi/) of revisions to the protocol specification, and it is time to revisit the topic. Given the substantial demand for code to experiment with, we are sharing the Windows Communication Foundation server and Silverlight client prototype implementation of one of the latest proposed drafts of the WebSockets protocol: [draft-montenegro-hybi-upgrade-hello-handshake-00](http://tools.ietf.org/id/draft-montenegro-hybi-upgrade-hello-handshake-00.txt).

You can read more about the effort and download the .NET prototype code at the [HTML5 Labs site](http://html5labs.interoperabilitybridges.com/prototypes/available-for-download/websockets). 

### What is WebSockets?

WebSockets is one of the HTML 5 working specifications driven by the W3C to define a duplex communication protocol for use between web browsers and servers. The protocol enables applications that exchange messages between the client and the server with communication characteristics that cannot be met with the HTTP protocol. 

In particular, the protocol enables the server to send messages to the client at any time after the WebSockets connection has been established and without the HTTP protocol overhead. This contrasts WebSockets with technologies based on the HTTP long polling mechanism available today. 

For this early WebSockets prototype we are using a Silverlight plug-in on the client and a WCF service on the server. In the future, you may see HTML5 Labs using a variety of other technologies. 

### What are we making available?

Along with the [downloadable .NET prototype](http://html5labs.interoperabilitybridges.com/prototypes/available-for-download/websockets/html5protos_Download) implementation of the WebSocket proposed [draft specification](http://tools.ietf.org/id/draft-montenegro-hybi-upgrade-hello-handshake-00.txt), we are also hosting a sample web chat application based on that prototype in Windows Azure at [http://html5labs.cloudapp.net/WebSockets/ChatDemo/wsdemo.html](http://html5labs.cloudapp.net/WebSockets/ChatDemo/wsdemo.html). The sample web chat application demonstrates the following components of the prototype:   

1. The server side of the WebSocket protocol implemented using Windows Communication Foundation from .NET Framework 4. The WCF endpoint the sample application communicates with implements the draft WebSocket [proposal](http://tools.ietf.org/id/draft-montenegro-hybi-upgrade-hello-handshake-00.txt).  
2. The client side prototype implementation consisting of two components:      

1. A Silverlight 4 application that implements the same draft of the WebSocket protocol specification.  
2. A jQuery extension that dynamically adds the Silverlight 4 application above to the page and creates a set of JavaScript WebSocketDraft APIs that delegate their functionality to the Silverlight application using the HTML bridge feature of Silverlight.  


The [downloadable package](http://html5labs.interoperabilitybridges.com/prototypes/available-for-download/websockets/html5protos_Download) contains a .NET prototype implementation consisting of the following components: 

1. A WCF 4.0 server side binding implementation of the WebSocket specification [draft](http://tools.ietf.org/id/draft-montenegro-hybi-upgrade-hello-handshake-00.txt).  
2. A prototype of the server side WCF programming model for WebSockets.  
3. Silverlight 4 client side implementation of the protocol.  
4. .NET 4.0 client side implementation of the protocol.  
5. A HTML bridge from the Silverlight to JavaScript that enables use of the prototype from JavaScript applications running in browsers that support Silverlight.  
6. Web chat and stock quote samples.  


Given the prototype nature of the implementation, the following restrictions apply: 

1. A Silverlight client (and a JavaScript client, via the HTML bridge) can only communicate using the proposed WebSocket protocol using ports in the range 4502-4534 (this is related to [Network Security Access Restrictions](http://msdn.microsoft.com/en-us/library/cc645032(v=vs.95).aspx) applied to all direct use of sockets in the Silverlight platform).  
2. Only text messages under 126 bytes of length (UTF-8 encoded) can be exchanged.  
3. There is no support for web proxies in the client implementation.  
4. There is no support for SSL.  
5. Server side implementation limits the number of concurrent WebSocket connections to 5.  


This implementation has been tested to work on Internet Explorer 8 & 9.

### Why is this important?

Through access to emerging specifications like WebSockets, the HTML5 Labs sandbox gives you implementation experience with the draft specifications, helps enable faster iterations around Web specifications without getting locked in too early with a specific draft, and gives you the opportunity to provide feedback to improve the specification. This unstable prototype also has the potential to benefit a broad audience. 

### We want your feedback

As you try this implementation we [welcome your feedback](http://mailinglist.interoperabilitybridges.com/scripts/wa-INTEROP.exe?A0=HTML5_WEBSOCKETS) and we are looking forward to your comments! }