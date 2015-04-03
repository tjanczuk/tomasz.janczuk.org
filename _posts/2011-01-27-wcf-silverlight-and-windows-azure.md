---
layout: post
title: WCF, Silverlight, and Windows Azure WebSocket samples
date: '2011-01-27T12:16:00.001-08:00'
author: Tomasz Janczuk
tags:
- JavaScript
- HTTP
- Websockets
- Silverlight
- web
- jQuery
- WCF
- browser
modified_time: '2011-01-27T12:54:40.945-08:00'
blogger_id: tag:blogger.com,1999:blog-2987032012497124857.post-4170799882175777871
blogger_orig_url: http://tomasz.janczuk.org/2011/01/wcf-silverlight-and-windows-azure.html
---




In my recent post [we announced a prototype implementation of WebSockets protocol in Windows Communication Foundation (WCF](http://tomasz.janczuk.org/2010/12/websockets-wcf-service-silverlight-and.html)). The implementation is available for download in binary form from the [HTML5Labs site](http://html5labs.interoperabilitybridges.com/prototypes/available-for-download/websockets). Since then many people expressed interest in the source code for the web chat sample we have shown as part of the prototype (and which is [deployed to Windows Azure](http://html5labs.cloudapp.net/WebSockets/ChatDemo/wsdemo.html)). You can now [download the code for the WebSockets samples](http://janczuk.org/code/samples/websocketssamples.zip) (chat, stock quotes, echo, along with Windows Azure), and read more about the programming model below.   

Compiling the samples requires Visual Studio 2010, WebSockets prototype binary drop, and Windows Azure SDK .  

### Server side  

The prototype implementation of WebSockets protocol provides a server side programming model in WCF that represents a WebSocket connection to a server using the WebSocketService class:

{% highlight csharp linenos %}
public abstract class WebSocketsService : IWebSockets, IDisposable
{
    protected WebSocketsService() {}
    protected Uri HttpRequestUri { get; }
    protected WebHeaderCollection HttpRequestHeaders { get; }
    public void Dispose() {}
    public virtual void OnOpen() {}
    public virtual void OnMessage(JsonValue jsonValue) {}
    public void Send(JsonValue jsonValue) {}
    protected void Close() {}
    protected virtual void OnError(object sender, EventArgs e) {}
    protected virtual void OnClose(object sender, EventArgs e) {}
    protected virtual void Dispose(bool disposing) {}
}

{% endhighlight %}



When a new WebSocket connection is made to the server, the OnOpen method is invoked. This gives the application an opportunity to inspect the HttpRequestUri as well as HttpRequestHeaders of the incoming HTTP Upgrade request to decide if it wants to accept or reject it (in which case OnOpen should throw an exception), as well as extract and store any information from the HTTP request. For example, the application could extract the resource name like a stock ticker from the URL, or an authentication token from an HTTP header. 

After the connection is established, OnMessage method is invoked every time a WebSocket message arrives from the client. At the same time, the application can call the Send method to asynchronously send a message to the client. The notable element is the use of JsonValue to represent messages exchanged between the client and the server. JsonValue provides a dynamic programming model for JSON data. I have described this mechanism in more detail in [my previous post about JsonValue](http://tomasz.janczuk.org/2010/10/wcf-support-for-jquery-on.html). 

A typical application would derive from WebSocketService to implement application-specific logic of handing WebSocket connections. An example could be a ChatService. The application can start listening for incoming WebSocket connections using the WebSocketsHost class: 

{% highlight csharp linenos %}
var sh = new WebSocketsHost<ChatService>(
    new Uri("ws://" + Environment.MachineName + ":4502/chat"));
sh.AddWebSocketsEndpoint();
sh.Open();

{% endhighlight %}



The prototype implementation of WebSockets in WCF does not support hosting of WebSocket WCF services in IIS. It means the service must be hosted in a console application or a Windows Service. Another implication of this limitation is that deploying a WCF WebSocket service to Windows Azure requires a worker role as opposed to a web role. 

### Client side

The client side implementation of the WebSockets protocol proposal is done in Silverlight and offers a WebSocket class modeled after [the proposed W3C JavaScript API for WebSockets](http://www.w3.org/TR/websockets/):

{% highlight csharp linenos %}
public class WebSocket : IDisposable
{
    public WebSocket() {}
    public WebSocket(string url) {}
    public event EventHandler<EventArgs> OnOpen;
    public event EventHandler<WebSocketEventArgs> OnMessage;
    public event EventHandler<EventArgs> OnClose;
    public string Url { get; set; }
    public WebSocketState ReadyState { get; }
    public Exception LastError { get; }
    public int MaxInputBufferSize { get; set; }
    public SynchronizationContext DispatchSynchronizationContext { get; set; }
    public void Open() {}
    public void Send(JsonValue value) {}
    public void Send(string data) {}
    public void Close() {}
    public void Dispose() {}
    protected virtual void Dispose(bool disposing) {}
}

{% endhighlight %}



The Open method establishes a WebSocket connection to a URL specified in the constructor or via the Url property. OnOpen event is raised when the WebSocket handshake has successfully completed. At the same time, the ReadyState property indicates the websocket is connected. At that point the application can start asynchronously sending messages to the server using the Send method. Messages received from the server are dispatched to the OnMessage event handler using the SynchronizationContext configured with the DispatchSynchronizationContext property. It provides a convenient mechanism to decide whether the incoming messages should be processed on the UI thread or a worker thread. 

Given that the prototype implementation of WebSockets in Silverlight is built on top of System.Net.Socket in Silverlight 4, it is subject to the same limitations: only ports in the range 4502-4534 can be used, and there is no support for SSL. 

The second client side component included in the WebSocket prototype is a jQuery plug-in that enables JavaScript browser applications to utilize the WebSocket functionality by creating a JavaScript WebSocket API that delegates to the Silverlight plug-in using HTML bridge functionality. The jQuery plug-in is implemented in the jquery.slws.js file included with the samples. It provides a $.slws.ready function that takes a delegate to be called only after the WebSocket functionality has been added to the environment (which requires the Silverlight application to be dynamically downloaded and injected into the current page):

{% highlight javascript linenos %}
$.slws.ready(function () {
    var connection = new WebSocketDraft('ws://' + window.location.hostname + ':4502/chat');
    connectin.onopen(function () {
        connection.send(JSON.stringify("Hello, World!"));
    });
    connection.onmessage(function (event) {
         var message = JSON.parse(event.data);
        alert(message);
    });
}

{% endhighlight %}



You can read more about this mechanism at [Mike Taulty’s blog](http://mtaulty.com/CommunityServer/blogs/mike_taultys_blog/archive/2010/07/27/silverlight-and-websockets.aspx).   