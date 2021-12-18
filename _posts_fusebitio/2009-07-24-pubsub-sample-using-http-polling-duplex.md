---
post_title: Pub/sub sample using HTTP polling duplex WCF channel in Microsoft Silverlight 3
date: 2009-07-24
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: pub/sub-sample-using-http-polling-duplex-wcf-channel-in-microsoft-silverlight-3
post_date_in_url: true
post_og_image: site
post_excerpt: Tomek on Software - shaken, not stirred
tags:
  - post
---




**NOTE** Many people reading this post are really looking for a publish\subscribe solution for Silverlight to implement a web chat or a stock quote application for the browser, a multiplayer online game, or some form of a web based collaboration tool, among others. If that fits your profile, you may want to check out http://laharsub.codeplex.com. Laharsub is an open source pub\sub message server for web clients, including Silverlight. It aspires to address the needs of people like you, who are interested in the subject matter of this post as well as willing to contribute your scenarios and requirements by leaving a comment. Enjoy!   

This sample demonstrates how a Microsoft Silverlight 3 application can consume asynchronous data notifications from the server using the polling duplex protocol of Windows Communication Foundation (WCF). I am also discussing selected aspects of the implementation, in particular related to performance. The [full source code and Visual Studio solution of the sample](http://janczuk.org/code/samples/PollingDuplexSample.zip) is available for download. Please also check the [Silverlight development environment prerequisites](http://silverlight.net/GetStarted/).  

The application follows a publisher/subscriber architecture and consists of three components:   

* PubSubService.svc is a WCF web service that manages client subscriptions to topics and sending asynchronous notifications to clients when data is published to the topic they are subscribed to. The service exposes an endpoint with a binding based on the polling duplex protocol. The server side implementation of the protocol is available through the PollingDuplexHttpBinding class in the System.ServiceModel.PollingDuplex.dll .NET Framework 3.5 library shipped as part of Microsoft Silverlight 3 SDK.  
* PubSubClient.aspx is an ASP.NET page that contains a Microsoft Silverlight 3 client application that can act as both a subscriber and a publisher. The application allows the user to subscribe to notifications related to a particular topic as well as publish data to that topic. The client communicates with the PubSubService.svc service using a WCF proxy and the client side implementation of the PollingDuplexHttpBinding, which ships as a Silverlight 3 extension assembly System.ServiceModel.PollingDuplex.dll within the Microsoft Silverlight 3 SDK.  
* PublisherClient.aspx is an ASP.NET page that contains a Microsoft Silverlight 3 client application that can act as a publisher only. The application allows the user to start publishing data to a particular topic at specified frequency, therefore causing a stream of regular notifications to be sent to subscribed clients. The client communicates with the backend PubSubService.svc using the same mechanism as PubSubClient.aspx.  
  

There are two key scenario types that can be demonstrated using this sample: collaboration and broadcast.   

#### Collaboration Scenario  

In the collaboration scenario we have a small set of clients (typically 2) acting as both publishers and subscribers of the same topic. An example of a collaboration scenario is a chat. In order to simulate the collaboration scenario, open the PubSubClient.aspx client application in two or more browser windows, subscribe to the same topic in each of them, and then start publishing to the topic by typing text in the provided text box in any of the clients. You should observe the text being propagated through notifications to all other clients.    

#### Broadcast Scenario  

In the broadcast scenario we have multiple clients subscribed to the same topic, and a small number of publishers (typically 1) publishing to the topic. An example of a broadcast scenario is propagating stock quotes from a single source to multiple client applications. In order to simulate the broadcast scenario, open the PubSubClient.aspx client application in multiple browser windows and subscribe to the same topic. Then open the PublisherClient.aspx client application in a single browser window and start publishing to the same topic you have subscribed to. You should observe periodic notifications arriving at all subscribed clients.     

#### Key implementation aspects  

##### Server  

The PubSubService.svc encapsulates the pub/sub server logic. It is a WCF duplex service exposed over the PollingDuplexHttpBinding that shipped with Microsoft Silverlight 3. The binding implements a comet-style long polling protocol which enables firewall traversal by leveraging HTTP protocol to enable duplex communication.   

A few implementation aspects of the service are worth calling out given their implications for performance of the service.   

In the broadcast scenario or a collaboration scenario that involves more than 2 participants, identical messages are often sent to several clients. Given that a substantial portion of the cost of sending a message is related to serializing its content, it is worthwhile to pre-serialize a message once and then send a copy of it multiple times. In order to accomplish this, the callback contract of a pub/sub service should take a Message as a parameter as opposed to typed parameters. This allows the message to be pre-serialized and converted to a MessageBuffer using TypedMessageConverter. Then, for every notification to be sent, the MessageBuffer can be used to create a clone of the Message without incurring the serialization cost. This is how the WCF service contract of the pub/sub service looks like:  

{% highlight csharp linenos %}


[ServiceContract(CallbackContract = typeof(INotification))]        
public interface IPubSub         
{         
    [OperationContract(IsOneWay = true)]         
    void Subscribe(string topic);     

    [OperationContract(IsOneWay = true)]        
    void Publish(string topic, string content);         
}         
        
[ServiceContract]         
public interface INotification         
{         
    [OperationContract(IsOneWay = true,         
                                AsyncPattern = true, Action="*")]         
    IAsyncResult BeginNotify(Message message,         
                                       AsyncCallback callback, object state);         
    void EndNotify(IAsyncResult result);         
}         
    

[MessageContract]        
public class NotificationData         
{         
    public const string NotificationAction = "[http://…";](http://..";)    

    [MessageBodyMember]        
    public string Content { get; set; }         
} 

{% endhighlight %}

  

When a notification is to be sent to multiple clients over the INotification service contract, the message is first pre-serialized using a TypedMessageConverter:  

{% highlight csharp linenos %}


TypedMessageConverter messageConverter =    
    TypedMessageConverter.Create(         
        typeof(NotificationData),         
        NotificationData.NotificationAction,         
        "[http://…");](http://..");)         
        
Message notificationMessage = messageConverter.ToMessage(         
    new NotificationData { Content = “hello” });         
MessageBuffer notificationMessageBuffer =         
    notificationMessage.CreateBufferedCopy(65536); 

{% endhighlight %}

  

Then a copy of the message is sent to all clients, thus avoiding the serialization cost on every send:  

{% highlight csharp linenos %}


foreach (INotification callbackChannel in clientsToNotify)        
{         
    try         
    {         
        callbackChannel.BeginNotify(         
            notificationMessageBuffer.CreateMessage(),    
            onNotifyCompleted, callbackChannel);         
    }         
    catch (CommunicationException) { }         
} 

{% endhighlight %}

  

Another aspect to emphasize in how the notifications are sent is related to the use of asynchronous API INotification.BeginNotify/EndNotify. Sending a large number of notifications is a high latency activity. Using asynchronous APIs to do so results in more efficient use of system resources compared to the use of synchronous APIs. Alternatively, synchronous APIs could be used if each of the notifications was scheduled to be sent on a separate worker thread from the thread pool. Measurements of both methods indicate the performance difference between them is negligible.   

Finally, the concurrency mode of the WCF service is set to ConcurrencyMode.Multiple, and instance mode to InstanceMode.Single. This requires explicit synchronization code to be added around access to critical resources (i.e. data structures related to subscriptions), but the extra effort pays off in reduced contention of concurrent requests to the service:  

{% highlight csharp linenos %}


[ServiceBehavior(        
    ConcurrencyMode = ConcurrencyMode.Multiple,    
    InstanceContextMode = InstanceContextMode.Single)]         
public class PubSubService : IPubSub         
{         
    // …          
} 

{% endhighlight %}

  

There are several more performance considerations for setting up a pub/sub service based on the HTTP polling duplex protocol. I will cover them in an upcoming post dedicated to performance tuning of such scenario.   

##### Client  

The key implementation aspect of the pub/sub service client in the sample is the use of the Add Service Reference feature of Visual Studio to automatically generate a proxy to such a service. This is a major usability improvement from Silverlight 2 to Silverlight 3. After the proxy has been added to the client project using the Add Service Reference feature of Visual Studio (or the slsvcutil.exe command line tool offering corresponding functionality), consuming asynchronous notifications from the sever is as easy as hooking up events on the generated proxy:  

{% highlight csharp linenos %}
   

PubSubClient client = new PubSubClient(        
    new PollingDuplexHttpBinding(),         
    new EndpointAddress("http://…"));         
this.client.NotifyReceived +=          
    new EventHandler<NotifyReceivedEventArgs>(NotifyReceived);           
this.client.SubscribeAsync("my topic");  

{% endhighlight %}

  

Thanks to this feature, the entire pub/sub client application is around 100 lines of C# code.   }