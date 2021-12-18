---
post_title: AJAX client for HTTP polling duplex WCF channel in Microsoft Silverlight 3
date: 2009-08-05
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: ajax-client-for-http-polling-duplex-wcf-channel-in-microsoft-silverlight-3
post_date_in_url: true
post_og_image: site
post_excerpt: Tomek on Software - shaken, not stirred
tags:
  - post
---




This sample shows how an AJAX browser application can receive asynchronous data notifications from a WCF service exposed using the HTTP polling duplex protocol that shipped in Microsoft Silverlight 3. The sample code contains a reusable, standalone JavaScript library (sl3duplex.js) which implements the client side of the polling duplex protocol. The [full source code and Visual Studio solution](http://janczuk.org/code/samples/PollingDuplexSampleAjax.zip) is available for download. Please also check the [Silverlight development environment prerequisites](http://silverlight.net/GetStarted/).  

This AJAX sample extends [the Silverlight-only pub/sub polling duplex sample](http://tomasz.janczuk.org/2009/07/pubsub-sample-using-http-polling-duplex.html) I published previously with the following components:  

* sl3duplex.js is a reusable, stand-alone JavaScript library that implements the client side of the HTTP polling duplex protocol compatible with the PollingDuplexHttpBinding in System.ServiceModel.PollingDuplex.dll (a .NET 3.5 library) that shipped in Microsoft Silverlight 3 SDK. The JavaScript library provides the Sl3DuplexProxy class that allows the user to asynchronously send messages to the server and receive asynchronous notifications. The library does not make any assumptions about the service contract (message structure) – it is a conceptual equivalent of IDuplexSession channel in WCF. The library has been tested with Internet Explorer 8, Firefox 3.5.2, and Chrome 2.0.  
* pubsub.js contains classes that represent pub/sub messages sent and received by the pub/sub WCF service in the sample. The classes abstract away the serialization/deserialization of data into XML and are meant to be used in conjunction with the Sl3DuplexProxy. This library is specific to the sample and not intended for direct reuse (meaning I did spend as much time on its quality as on sl3duplex.js).  
* PubSubClientAjax.htm is an AJAX client application that uses the two libraries above to implement a pub/sub client. The functionality of the client is identical to PubSubClient.aspx, a Silverlight 3 version that was discussed in detail in [the Silverlight-only pub/sub polling duplex sample](http://tomasz.janczuk.org/2009/07/pubsub-sample-using-http-polling-duplex.html).  
  

The Default.aspx page in the sample  contains detailed instructions on how to run it. Below I am going to discuss some of the implementation aspects of the AJAX client.     

### Sl3DuplexProxy class  

The Sl3DuplexProxy class from sl3duplex.js implements the client side of the HTTP polling duplex protocol compatible with PollingDuplexHttpBinding in Microsoft Silverlight 3. It enables client AJAX application to receive asynchronous notifications from a WCF backend service exposed over the polling duplex binding. The class offers a handful of methods and is simple to use.   

The constructor of the class allows the user to specify the URL of the backend WCF service, an event handler to be invoked when an asynchronous message arrives from the server, and an event handler to be invoked when an error occurs:   

{% highlight text linenos %}


org.janczuk.sl3duplex.Sl3DuplexProxy = function(args /* .url, .onMessageReceived, .onError */) 

{% endhighlight %}

  

The PubSubClientAjax.htm uses the constructor in the following way:  

{% highlight text linenos %}


proxy = new sl3duplex.Sl3DuplexProxy({       
    url: window.location.href.substring(0, window.location.href.lastIndexOf("/")) + "/PubSubService.svc",        
    onMessageReceived: function(body) {        
        addNotification("SERVER NOTIFICATION: " + (new sl3duplex.NotificationMessage(body)).text);        
    },        
    onError: function(args) {        
        addNotification("ERROR: " + args.error.message);        
        alignUIWithConnectionState(false);        
    }        
}); 

{% endhighlight %}

  

The URL to the backend service is constructed relative to the URL of the current page.   

The onMessageReceived function is invoked every time an asynchronous notification is received from the WCF backend. It accepts one parameter, which is a DOM Element object representing the Body of the SOAP envelope received from the server. In this implementation, the client is using the NotificationMessage helper class from pubsub.js to extract the string content of the message and display it in a textarea of the HTML page.   

The onError function is called when a communication or protocol error occurs. The proxy instance is always faulted on error and cannot be used for sending or receiving messages. The function accepts one parameter, which is an object with the following properties:  

* error is an instance of Error that contains the description of the immediate problem  
* httpRequest is an instance of XMLHttpRequest related to the error  
* proxy is an instance of the Sl3DuplexProxy  
* message (optional) represent the content of the message the client attempted to send to the server and is present when the error occurred as a result of the call to Sl3DuplexProxy.send method  
  

After the proxy has been created, the send method can be used to asynchronously send a message to the WCF backend:  

{% highlight text linenos %}


org.janczuk.sl3duplex.Sl3DuplexProxy.prototype.send = function(args /* .message.action, .message.body, .onSendSuccessful */) 

{% endhighlight %}

  

 The first call to send on a proxy creates a new session on the server which in turn enables the client to start receiving notification from the server. As a corollary, the client proxy will not receive any notifications from server before the first call to the send method. The send method allows the user to provide the SOAP action of the message (.message.action), the content of the SOAP Body represented as a string (.message.body), and a handler to be invoked when the server confirms successful reception of the message (.onSendSuccessful).   

The PubSubClientAjax.htm uses the send method in the following way:  

{% highlight text linenos %}


proxy.send({ message: new sl3duplex.SubscribeMessage(topic),       
             onSendSuccessful: function(args) {        
                 addNotification("CLIENT ACTION: Subscribed to topic " + topic);        
             }        
           }); 

{% endhighlight %}

  

The message object is created using the SubscribeMessage helper class from pubsub.js, which specifies the SOAP action value for that message as well as provides primitive encoding of a string topic name into an XML chunk that will be included in the SOAP Body (the encoding is primitive because the string is embedded “as is”, without proper XML encoding).   

The onSendSuccessful is called when the server confirms successful reception of the message with an HTTP 200 response. The function accepts one parameter, which is an object with the following properties:  

* httpRequest is an instance of XMLHttpRequest used to send the message and receive the confirmation  
* proxy is an instance of the Sl3DuplexProxy  
* message is the message instance passed to the send method  
  

Note that when an error occurs when sending the message, the onError handler specified in Sl3DuplexProxy constructor will be called.  

When the application is done using the proxy instance (wants to stop receiving notifications), the close method must be explicitly called:   

{% highlight text linenos %}


org.janczuk.sl3duplex.Sl3DuplexProxy.prototype.close = function() 

{% endhighlight %}

  

Until the close method is called, the proxy will continue issuing HTTP long polls to the server, hence taking up one connection out of the connection pool to the backend, and generating unnecessary network traffic. More details on the actual protocol and connection use are in he subsequent section. The close method can be called any number of times.   

The Sl3DuplexProxy instance maintains a notion of its state which can be queried with the getStatus method:  

{% highlight text linenos %}


org.janczuk.sl3duplex.Sl3DuplexProxy.prototype.getStatus = function() 

{% endhighlight %}

  

The function returns a numeric value representing the state. Sl3DuplexProxy defines defines instance properties of Created, Active, Closed, and Faulted that capture bit masks for inspecting the state of the proxy. Possible proxy states and state transitions are as follows:  

* Created – constructor has been called but the send method has not been called yet. This means the proxy is not receiving any notifications from the server. Calling send method will transition the proxy to Active or Closed \| Faulted state, depending on whether the send was successful. Calling close method will transition the proxy to the Closed state.  
* Active – in this state the proxy may receive notifications from the server. It also means the client AJAX application is performing active HTTP polling. Calling close method in this state will transition the proxy to the Closed state. A communication error as a result of calling send or receiving an erroneous notification from the server will transition the proxy to the Closed \| Faulted state.  
* Closed – in this state the proxy does not receive notifications and cannot be used for sending messages to the server  
* Faulted – an error occurred and the proxy was faulted. A faulted proxy is also in the Closed state, with the same implications for sending and receiving messages.  
  

Sl3DuplexProxy code has been tested with Internet Explorer 8, Firefox 3.5.2, and Chrome 2.0.   

### Polling duplex protocol considerations in AJAX  

The implementation of the polling duplex protocol in sl3duplex.js has a few interesting traces and limitations.  

An application can create many Sl3DuplexProxy instances to different WCF services on the same host (scheme/hostname/port combination) at any point in time. Specifically, creating proxies to services on different hosts at the same time is not supported by the implementation. For example, an application can create two instances of Sl3DuplexProxy, one for http://foo.com:1234/abc.svc, and the other for http://foo.com:1234/def.svc, but not for http://foo.com:1234/abc.svc and http://baz.com:1234/abc.svc. This limitation should not affect any practical scenario, since the underlying XMLHttpRequest stack only supports communication with the origin server of the page without breaching default cross-domain security setting of a browser.   

The implementation will start long HTTP polling the moment the send method is called on one of the created proxies. Regardless how many proxy instances are created, there is only one active poll request at any point in time. This means that an AJAX application will use only up to one HTTP connection from the browser pool to perform the HTTP polling at any time, regardless of the number of proxy instances. Note that any messages send asynchronously to the server using the send method will use an additional connection, but the latency of these calls should be low (i.e.  the WCF server side implementation immediately responds to these HTTP request with an empty HTTP 200 response.   

The polling continues in the background of an AJAX application as long as there is at least one Sl3DuplexProxy instance in the Active state, so care must be taken to call the close() method on a proxy instance once it has served its purpose.   

Using only one connection for polling requires a mechanism to demux server notifications to a client proxy the message is intended for. This is achieved with a notion of a session identifier. Every instance of Sl3DuplexProxy has its own unique session identifier created on instantiation, which can be queried with the getSessionId method:  

{% highlight text linenos %}


org.janczuk.sl3duplex.Sl3DuplexProxy = function(args /* .url, .onMessageReceived, .onError */) {       
    this.getSessionId = (        
        function() {        
            var sessionId = org.janczuk.sl3duplex._createGuid();        
            return function() { return sessionId; };        
        }        
    )(); 

{% endhighlight %}

  

Session identifier of the client proxy is included in all messages sent from that proxy to the server as well as notifications from the server intended for that particular client. This enables demux of notifications received over a single polling connection to the appropriate client proxy instance.  }