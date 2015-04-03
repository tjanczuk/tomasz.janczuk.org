---
layout: post
title: Silverlight HTML5 WebSocket client with an HTML bridge to Ajax/JavaScript
date: '2010-07-20T15:40:00.000-07:00'
author: Tomasz Janczuk
tags:
- Websockets
- Silverlight
- jQuery
- WCF
modified_time: '2010-12-21T09:35:29.493-08:00'
blogger_id: tag:blogger.com,1999:blog-2987032012497124857.post-2871885842393034810
blogger_orig_url: http://tomasz.janczuk.org/2010/07/silverlight-html5-websocket-client-with.html
---




**NOTE** Answering to popular demand expressed in comments to this thread, we are now making a .NET prototype implementation of a WebSocket draft specification available for download. Check out the details at http://tomasz.janczuk.org/2010/12/websockets-wcf-service-silverlight-and.html.   

One of the HTML 5 initiatives is to define a duplex communication protocol called WebSocket for use between web browsers and servers. The protocol enables a number of applications that must exchange messages between the client and the server with performance characteristics that cannot be met with the HTTP protocol. In particular, the protocol enables the server to send messages to the client at any time after the WebSocket connection has been established and without the HTTP protocol overhead. This contrasts WebSocket to technologies based on the HTTP long polling mechanism available today.      
A sample web chat application built using a prototype implementation of the WebSocket protocol is available at [http://40interop.ep.interop.msftlabs.com/html5/wschat.html](http://40interop.ep.interop.msftlabs.com/html5/wschat.html), as well as hosted in Windows Azure at [http://websockets.cloudapp.net/wsdemo.html](http://websockets.cloudapp.net/wsdemo.html). The prototype consists of the following components:       
  

1. The server side of the WebSocket protocol implemented using Windows Communication Foundation (WCF) from .NET Framework 4. The endpoint at ws://40interop.ep.interop.msftlabs.com:4502/servicemodelsamples/chat.svc implements two currently considered protocol proposals: the [draft-ietf-hybi-thewebsocketprotocol-00](http://www.ietf.org/id/draft-ietf-hybi-thewebsocketprotocol-00.txt) (also referred to as version 76), and [draft-hixie-thewebsocketprotocol-75](http://tools.ietf.org/html/draft-hixie-thewebsocketprotocol-75). The server implementation automatically detects the version the client is using.  
2. The client side of the WebSocket protocol implementation consists of two components:      

1. A Silverlight 4 application that implements the [draft-hixie-thewebsocketprotocol-75](http://tools.ietf.org/html/draft-hixie-thewebsocketprotocol-75) version of the WebSocket protocol. It is available at [http://40interop.ep.interop.msftlabs.com/html5/ClientBin/Microsoft.ServiceModel.Websockets.xap](http://40interop.ep.interop.msftlabs.com/html5/ClientBin/Microsoft.ServiceModel.Websockets.xap).  
2. A jQuery extension that determines if the web browser in which the JavaScript application is running supports the WebSocket protocol natively or not. If the browser supports the WebSocket protocol natively, the jQuery extension gets out of the way. If the browser does not support the WebSocket protocol natively but does support Silverlight 4, the jQuery extension dynamically adds the Silverlight 4 application (#1) to the page and creates a set of JavaScript WebSocket APIs that delegate their functionality to the Silverlight application using the HTML bridge feature of Silverlight. It is available at [http://40interop.ep.interop.msftlabs.com/html5/js/jquery.slws.js](http://40interop.ep.interop.msftlabs.com/html5/js/jquery.slws.js).  
  

Given the approach taken, the sample chat application based on WebSockets should be working in all major browsers (Internet Explorer, Firefox, Chrome, Safari, Opera).    
      
Mike Taulty has an [excellent post](http://mtaulty.com/CommunityServer/blogs/mike_taultys_blog/archive/2010/07/27/silverlight-and-websockets.aspx) explaining in more detail the client side implementation.   