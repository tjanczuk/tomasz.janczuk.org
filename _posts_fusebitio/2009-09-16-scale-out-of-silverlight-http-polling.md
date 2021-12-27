---
tags: ['post']
post_og_image: 'site'
date: '2009-09-16'  
post_title: Scale-out of Silverlight HTTP polling duplex WCF service in a web farm scenario
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: scale-out-of-silverlight-http-polling-duplex-wcf-service-in-a-web-farm-scenario
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




### Introduction  

In my recent articles I have introduced a sample [pub/sub application using Silverlight’s HTTP polling duplex protocol](http://tomasz.janczuk.org/2009/07/pubsub-sample-using-http-polling-duplex.html), an [AJAX client talking the same protocol](http://tomasz.janczuk.org/2009/08/ajax-client-for-http-polling-duplex-wcf.html), as well as discussed [server side performance of the protocol](http://tomasz.janczuk.org/2009/08/improving-performance-of-concurrent-wcf.html). The latter post pointed out scalability challenges associated with deploying a WCF service using the Silverlight HTTP polling duplex protocol in a web farm scenario. This article introduces one possible solution to the scale-out problem, enabling an applications utilizing the protocol to accommodate an arbitrary number of concurrent clients through scale-out of the backend infrastructure.   

The source of the scalability limitations of Silverlight 3 PollingDuplexHttpBinding on the server is the sessionful nature of the WCF channel the binding creates, which requires in-memory state to be maintained for the lifetime of the session. Specifically, there are two problems associated with hosting such a service in a practical scale-out scenario in a web farm:   

1. Backend affinity. Subsequent calls from the same client session must be directed to the same web farm backend by the network load balancer (NLB). Oftentimes (in particular in third-party hosted scenarios) the user does not have the level of control over the NLB configuration to ensure backend affinity.  
2. Backend recycling. An application hosted in a web farm should not rely on any transient state maintained on a backend instance between subsequent client calls – the backend can be recycled and recreated at any time between client calls. In particular, any in-memory state may be lost, including the in-memory WCF session state.  
  

The approach to solve these challenges described here suggests the WCF service handles the HTTP long polls explicitly and maintains the sessionful state in a scaled-out backend infrastructure, therefore avoiding any in-memory state that spans subsequent client calls on the web front-end , and not requiring backend affinity from the NLB. In order to describe this approach, one must first understand the mechanics of the Silverlight HTTP polling duplex protocol.   

### Silverlight HTTP polling duplex protocol  

The Silverlight HTTP polling duplex protocol has been designed to enable data push from the server to the client while optimizing for the web browser client environment. Web browsers typically enforce a limit on the number of concurrent TCP connections to a single web server to make denial of service attacks on that server harder. For example, Internet Explorer 8 by default allows 6 TCP connections (equal to the limit of concurrent HTTP requests) to a single backend host. A Silverlight application hosted in a web browser is subject to these limitations.   

A single Silverlight application enables multiple logically distinct duplex sessions to be established with a single backend using the WCF duplex session channel abstraction. A session in this protocol involves two independent mechanisms for sending messages from the client to service and from the service to the client. Messages from the client to service within a particular session are sent using individual, short lived HTTP requests, followed by empty HTTP responses from the server. Messages from the server to the client within a particular session are sent using an HTTP response to a long lasting HTTP request the client makes to the server (the “long poll”). The client side implementation of the polling duplex protocol ensures that the server always has such pending HTTP request available; as soon as the server sends a message to the client using the response to the long poll, the client issues a new long poll.   

One implication of the long polling mechanism is that at any point in time one TCP connection between the client and the server is dedicated to the long poll. If multiple duplex sessions between a Silverlight application and the server were to issue their individual long polls, the browser connection limit would quickly be reached, preventing other communication. Therefore, a single long poll request per backend server is shared across all logical duplex sessions within a single Silverlight application. This ensures that only one TCP connection from the browser connection limit is utilized for the polling duplex protocol at any time. The polling duplex protocol on the server side multiplexes the messages for multiple sessions over that single long poll. Similarly, the protocol on the client dispatches the messages received over the single long poll to appropriate logical session on the client. The protocol is based on the [WS-MakeConnection](http://docs.oasis-open.org/ws-rx/wsmc/200702) specification.   

 ![Silverlight HTTP Polling Duplex Protocol](http://lh4.ggpht.com/_NUp_nWDyyvI/SrF-lr90l4I/AAAAAAAABAk/F08EIfV94t8/pollingduplex_thumb%5B4%5D.png?imgmax=800)  

The server side of the protocol maintains an in-memory queue of messages to be sent to a particular client, abstracting the existence and handling of the long polls away from the application layer. While this allows the connection between the client and the server to be modeled as a WCF IDuplexSession channel on the server, it imposes the scalability issues associated with maintaining in-memory state. In a scenario where the server is scaled out to multiple instances behind a load balancer without backend affinity, it is possible that subsequent long polls from a single client will reach different instances of the backend. In particular, it may reach an instance that does not maintain the message queue for the client issuing the long poll. In addition, a recycled backed may loose the in-memory message queue.     

### Enabling scale out of the polling duplex backend service  

The approach to enabling scale out of a backend service using the polling duplex protocol relies on removing the polling duplex layer of the protocol from the server and relying on the application implementing equivalent logic directly on top of the HTTP protocol. In particular, this approach implies the application must explicitly handle the long polls from the client, understand the one-to-many relationship between a long poll and a logical session, as well as provide a scalable store for messages to be sent to the client(s) accessible from all instances of the backend.   

 ![Scaleable Silverlight HTTP Polling Duplex Protocol](http://lh6.ggpht.com/_NUp_nWDyyvI/SrF-nEMMIWI/AAAAAAAABAs/9BSV4eIMLDg/pollingduplex-simplex_thumb%5B2%5D.png?imgmax=800)   

These are the key implications of such a refactoring of the server:  

1. The service contract of the WCF service changes from duplex to simplex. The service is now exposed directly over HTTP and as such consists of request/response service operations.  
2. The service contract must contain an operation contract for handling the long polls from the client (let’s call it a “MakeConnect” operation following the naming from [WS-MakeConnection](http://docs.oasis-open.org/ws-rx/wsmc/200702) specification).  
3. All service operations except MakeConnect must be one way. This means the operations do not return any application level response to the client (an empty HTTP response is sent). If the service has any messages to send as a result of processing the client call, it needs to use the long polling processing logic to do so. Typically this would involve storing the message to be sent to the client in a scalable backend storage to be picked up by the long poll processing logic in due time (this is depicted as the dotted arrow on the diagram above).  
4. Given the the wire format of the protocol remains unaffected, existing clients continue to function without any changes.  
  

Doing all of the above may appear a daunting task at first, but with the appropriate choice of technologies on the backend and a few helper classes I am going to show it is not hard at all.     

  

### Show me the code  

I am going to use the pub\sub sample using the Silverlight HTTP polling duplex protocol I wrote about before as a starting point, and describe the steps that were necessary to convert the WCF backend to a scalable, web-farm friendly implementation. It is a good idea to [read about and run the original sample](http://tomasz.janczuk.org/2009/07/pubsub-sample-using-http-polling-duplex.html) to understand its functionality before moving on. The [Visual Studio solution containing the complete pub\sub sample after conversion](http://janczuk.org/code/samples/PollingDuplexSampleScalable.zip) is available for download.   

The converted VS solution contains a new project called Microsoft.ServiceModel.PollingDuplex.Scalable, which includes helper classes and methods that facilitate the conversion of an arbitrary duplex WCF service to a service exposed over the Silverlight polling duplex binding. The key components and conversion steps are described below.   

### Service contract change from duplex to simplex  

The IPollingDuplex service contract from Microsoft.ServiceModel.PollingDuplex.Scalable project enables explicit handling of long polling messages by the service implementation:  

```
[ServiceContract]       
public interface IPollingDuplex        
{        
    [OperationContract(        
        AsyncPattern = true,        
        Action = "[http://docs.oasis-open.org/ws-rx/wsmc/200702/MakeConnection"](http://docs.oasis-open.org/ws-rx/wsmc/200702/MakeConnection"),        
        ReplyAction = "*")]        
    IAsyncResult BeginMakeConnect(MakeConnection poll, AsyncCallback callback, object state);        
    Message EndMakeConnect(IAsyncResult result);        
}     

[MessageContract(WrapperName = "MakeConnection", WrapperNamespace = "[http://docs.oasis-open.org/ws-rx/wsmc/200702")]](http://docs.oasis-open.org/ws-rx/wsmc/200702")])        
public class MakeConnection        
{        
    [MessageBodyMember(Name = "Address", Namespace = "[http://docs.oasis-open.org/ws-rx/wsmc/200702")]](http://docs.oasis-open.org/ws-rx/wsmc/200702")])        
    public string Address { get; set; }        
    // …        
} 

```
  

  

The contract is intentionally asynchronous, as handling of long polls is by definition a blocking operation (and likely asynchronous itself depending on the technology used for storing the messages the server needs to send to the client). This contract enables the server to receive the long poll and respond with an arbitrary message back (the response must target one of the logical sessions associated with the client who made the long poll request).   

A service that wants to implement the polling duplex protocol in a scalable manner must have service contract derived from the IPollingDuplex service contract above. In addition, the application service contract must be simplex (as opposed to duplex). For example, this is the service contract for the pub\sub service from the original sample:  

```
[ServiceContract]       
public interface IPubSub : IPollingDuplex        
{        
    [OperationContract(IsOneWay = true)]        
    void Subscribe(string topic);     

    [OperationContract(IsOneWay = true)]       
    void Publish(string topic, string content);        
} 

```
  

  

Notice the callback contract is gone, as well as the Subscribe and Publish operations are one way: the notifications to clients are now sent in the form of a response to the IPollingDuplex.MakeConnect call.   

### Implementing MakeConnect  

IPollingDuplex.MakeConnect operation receives the MakeConnection message which contains the unique address of the client making the request in the MakeConnection.Address property. The implementation should determine if there are any notifications to be sent to ***any*** of the sessions associated with this client (the model assumes 1-N relationship between a client and a logical duplex session with the client, as described before). If there is a notification to be sent to one of the sessions, the response message should be constructed in two steps: first create an instance of the message with the desired action and payload:  

```
Message response = Message.CreateMessage(MessageVersion.Default, “uri:someaction”, somePayload); 

```
  

(The message format and action must of course match what the client expects to receive given its service contract).   

And then the polling duplex protocol SOAP headers need to be added to the message to indicate which of the sessions on the client this notification should be dispatched to. There is a helper method on the MakeConnection class to make it easy:  

```
MakeConnection poll;       
string sessionId;        
response = poll.PrepareRespose(response, sessionId); 

```
  

MakeConnect implementation should hold onto the request without responding for a specific time (e.g. 15 seconds) even if there are no notifications to be sent back to the particular client. If after that time there is still no notification scheduled to be sent for any of the sessions on the client, the implementation of MakeConnect should respond with an empty HTTP 200. This is made easy with a call to MakeConnection.CreateEmptyResponse():  

```
MakeConnection poll;       
Message response = poll.CreateEmptyResponse(); 

```
  

### Accessing session information from “application” service operations  

Application service operations other than IPollingDuplex.MakeConnect (e.g. Publish and Subscribe in this sample) must have access to the client and session information from the polling duplex protocol. For example, the IPubSub.Subscribe method needs this information to register a particular client+session to receive notifications related to a particular topic. The IPubSub.Publish method may need this information to avoid sending a notification back to the client+session that has just invoked the Publish method.   

The way to get at the session information from an incoming application message is to call an extension method on the OperationContext (the method is implemented in the Microsoft.ServiceModel.PollingDuplex.Scalable project):  

```
PollingDuplexSession incomingSession = OperationContext.Current.GetPollingDuplexSession(); 

```
  

The PollingDuplexSession contains two identifiers that uniquely identify the client and the session on the client:  

```
public class PollingDuplexSession       
{        
    public string Address { get; set; }        
    public string SessionId { get; set; }        
} 

```
  

  

The PollingDuplexSession.SessionId is the same identifier as the MakeConnection.PrepareResponse method requires (note that MakeConnection class already knows about the client’s address).   

### Exposing the service  

The simplex service can now be exposed directly over an HTTP binding as opposed to polling duplex sessionful binding:  

```
<system.serviceModel>       
  <bindings>        
    <customBinding>        
      <binding name="PubSub">         
        <httpTransport/>          
      </binding>        
    </customBinding>        
  </bindings>        
  <services>        
    <service name="Microsoft.Samples.Silverlight.PollingDuplex.Service.PubSubService">        
      <endpoint address="" binding="customBinding" bindingConfiguration="PubSub"        
       contract="Microsoft.Samples.Silverlight.PollingDuplex.Service.IPubSub" />        
    </service>        
  </services>        
</system.serviceModel> 

```
  

### Scalable server side storage for messages to be sent to the client  

The approach described here removes the in-memory queue the server side implementation of the polling duplex protocol maintains for storing messages the server needs to send to the client. Instead, the application code must now decide where these messages are stored and how the store is queried from within the IPollingDuplex.MakeConnect implementation. There are a few requirements the store technology must meet in order to ensure scalability:  

1. The store must be accessible from all scaled-out instances of the WCF service.  
2. The store should provide an asynchronous mechanism of querying for data to avoid blocking the thread on which the WCF service received the MakeConnect call. The entire purpose of making the IPollingDuplex contract asynchronous is to optimize system resource use during processing of the long poll, which is by definition a long running, blocking operation.  
  

The [sample code of the solution](http://janczuk.org/code/samples/PollingDuplexSampleScalable.zip) stops short of recommending a particular backing store. In fact, it uses a mock up of the store that is in-memory based, which means the sample would not even scale out as is. However, it provides a good demonstration of the refactoring steps specific to the protocol itself.   

[Aleksey Savateyev](http://blogs.msdn.com/ales/), a colleague of mine, has taken this sample to an entirely different level by prototyping a pub\sub solution that uses [Windows Azure Queues](http://msdn.microsoft.com/en-us/library/dd179363.aspx) as the backend storage, and allows scale-out of the polling duplex WCF service using the pattern described here to an arbitrary number of web roles in the [Windows Azure](http://www.microsoft.com/azure/default.mspx) platform. Aleksey promised to blog about this approach, so check out [Aleksey’s site](http://blogs.msdn.com/ales/).   }