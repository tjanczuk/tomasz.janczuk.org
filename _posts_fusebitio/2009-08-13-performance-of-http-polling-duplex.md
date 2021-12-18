---
post_title: Performance of HTTP polling duplex server-side channel in Microsoft
  Silverlight 3
date: 2009-08-13
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: performance-of-http-polling-duplex-server-side-channel-in-microsoft-silverlight-3
post_date_in_url: true
post_og_image: site
post_excerpt: Tomek on Software - shaken, not stirred
tags:
  - post
---




### Introduction  

Silverlight supports a communication protocol that allows a server to asynchronously send messages to a Silverlight client (“push”) using Windows Communication Foundation (WCF) services. This feature is based on a polling duplex (HTTP long polling, or Comet-style) protocol, and ships as two DLLs in the Microsoft Silverlight SDK, both named System.ServiceModel.PollingDuplex.dll – one for the Silverlight client and one for the server. More information can be found in [MSDN help](http://msdn.microsoft.com/en-us/library/dd470105(VS.95).aspx).   

In Silverlight 3, the usability of this feature was greatly improved, and integrated with the Add Service Reference tool in Visual Studio, making it very easy to write client-side code to consume messages “pushed” from the server.   

However, writing the server-side “push” service in a way that performs well for a large number of clients, and assessing the viability of the polling duplex protocol for any given application, remained a challenge. This post and the [accompanying sample](http://tomasz.janczuk.org/2009/07/pubsub-sample-using-http-polling-duplex.html) attempt to help you decide whether this technology is suitable for the requirements of your scenario.  

### Scenarios  

Scenarios that were considered for measuring polling duplex performance are based on the pub/sub architecture. Clients can contact the server to subscribe to events associated with a specific topic. When an event is published for that topic, the server sends out notification about that event to all clients who subscribed to the topic.   

Within this pub/sub architecture, two classes of scenarios were considered: broadcast and collaboration.  

#### Broadcast Scenario  

In this scenario, many clients subscribe to the same topic. When an event is published to that topic, all clients receive a notification.   

 ![broadcast](http://lh3.ggpht.com/_NUp_nWDyyvI/SoUCyb3EjCI/AAAAAAAAA98/_mbSpWL92Xs/broadcast_thumb%5B1%5D.png?imgmax=800)   

For example, if 3,000 clients are connected, such a scenario could involve all 3,000 clients monitoring the price of a certain stock, or the progress of a certain sporting event, through a Silverlight application. As soon as the stock quote or the sports score changes, the same notification is broadcast to all 3,000 clients.  

#### Collaboration Scenario  

In this scenario, several topics exist. Each topic has several clients acting as both subscribers and publishers. Upon publishing of an event for a topic by one of the clients, the server notifies all remaining clients subscribing to this topic of that event.   

 ![collaboration](http://lh5.ggpht.com/_NUp_nWDyyvI/SoUCzSwW78I/AAAAAAAAA-E/mJZmhAjObsU/collaboration_thumb%5B1%5D.png?imgmax=800)  

For example, if 3,000 clients are connected, such a scenario could involve 1,500 one-on-one technical support chats between a customer and a technician through a Silverlight chat client, or 1000 real-time games with 3 participants each, or 1,500 collaborations with 2 people editing the same diagram or data grid in real-time. In all these cases, the action of one person immediately gets “pushed” to their collaborator(s), but does not affect the other clients. In reality, many scenarios will fall in between the two extremes of broadcast and 2-client collaboration.   

### Scalability vs performance  

Current implementation of polling duplex protocol in Silverlight 3 requires client affinity to a particular physical server machine for the lifetime of the WCF channel (WCF proxy). Moreover, the server maintains in-memory state for the duration of the session with the client. If you are using a load balancer that cannot guarantee client affinity to a particular backend, or if your hosting infrastructure cannot guarantee that the service will keep running on the same machine, the protocol will fail. In the practice of load balanced web farms, the Silverlight polling duplex protocol in the current form does not scale out well.   

You can work around the backend affinity limitation by performing load-balancing manually at the application level. For example, suppose that for your scenario, based on this document or your own measurements, you discover that you can only support 3,000 clients, but need to support 6,000. You can set up two servers, with explicitly different domain names (e.g. at **service1.contoso.com** and **service2.contoso.com**), and the client can select one of these to connect to either randomly, based on a hash of a known value (e.g. topic name), or by calling a “discovery service” that returns the address of the duplex service to connect to.  

There is no easy way to work around the problem of the server maintaining in-memory session state at the moment. We are actively working on addressing this problem in future releases.   

Taking these scalability considerations into account, it is still important to understand performance characteristics of the protocol implementation to assess its suitability for a particular scenario. Performance is the main topic of this post.   

### Tuning  

To optimize the performance of the Polling Duplex component, certain settings in IIS, ASP.NET, the .NET Framework and WCF need to be adjusted. All measurements in this document assume that these modifications have been made. Some of the settings discussed below need to be changed when a system is put into production. Such differences are discussed when appropriate. Please refer to the [accompanying sample](http://tomasz.janczuk.org/2009/07/pubsub-sample-using-http-polling-duplex.html) for a additional tuning information.  

#### WCF configuration tuning  

Polling duplex protocol has been implemented on the server side as a WCF binding that provides session channels. Each session channel corresponds to a single client connection. The number of session channels that a WCF service can support concurrently is throttled using ServiceThrottlingBehavior.MaxConcurrentSessions service behavior. In order to measure the maximum number of connections the server can support, this throttle needs to be increased. For the purpose of this measurement, we increased the throttle to Int32.MaxValue using WCF configuration:  

{% highlight text linenos %}


<serviceThrottling maxConcurrentSessions="2147483647"/> 

{% endhighlight %}

  

When the system is put into production, you would want to set the value of maxConcurrentSessions to match the maximum number of clients your service can support concurrently.   

There are several customizations that were introduced in the settings of the PollingDuplexBindingElement for the purposes of this performance measurements:  

{% highlight text linenos %}


<binding name="PubSub">        
    <binaryMessageEncoding/>         
    <pollingDuplex maxPendingSessions="2147483647"         
                 maxPendingMessagesPerSession="2147483647"         
                 inactivityTimeout="02:00:00"         
                 serverPollTimeout="00:05:00"/>         
    <httpTransport/>         
</binding> 

{% endhighlight %}

  

The value of inactivityTimeout controls the maximum time without any activity on the channel before the channel is faulted. The value has been set to 2 hours to avoid a situation when a channel is faulted due to infrequent message exchanges in a test variation. In production, you should set this value to exceed the expected duration of a client connection, which is application specific. Regardless of the value though, the client code should take appropriate fault-tolerance measures to possibly re-establish a connection when the channel is faulted due to inactivity.   

The value of serverPollTimeout controls the maximum time the server will hold onto client’s long poll HTTP request before responding. If that time has elapsed without the server having an application message to push back to the client, the server will send back an empty HTTP OK response (causing the client to re-issue a new long poll). For the purpose of this measurement, the value has been set to 5 minutes to minimize the frequency and therefore cost of empty polls. The default value of this setting is 15 seconds. In production, in addition to performance consideration, one should consider the presence of proxy servers which may limit the duration of outstanding HTTP requests.   

The value of maxPendingSessions throttles the number of new sessions that wait to be accepted on the server. This situation can occur when the speed at which new sessions are established at the server (new clients connect) exceeds the server’s ability to accept them. This value has been increased to Int32.MaxValue from the default of 4 to allow for the client connection pattern implemented in our performance code, where all the clients attempt to connect at once at the beginning of the test. In a typical production scenario, new client connections are spread more evenly in time, in which case the default value of 4 should be adequate.   

The value of the maxPendingMessagesPerSession throttles the number of new messages from a client (sent over a particular session) that wait to be accepted by the server. Similarly to maxPendingSessions, this situation can occur when the speed at which new messages arrive at the server exceeds the server’s ability to accept them. The value has been increased to Int32.MaxValue (the default is 8) to allow for the initial wave of messages given the client connection pattern in the test. In a typical production scenario, messages from the client can be expected to be spread more evenly in time, in which case the default value of 8 should be appropriate.   

PollingDuplexHttpBinding standard binding by default uses binary encoding (newly added in Silverlight 3). The custom binding used in the performance measurements also uses binary encoding:  

>
> <binaryMessageEncoding/>  

Binary encoding has several performance benefits compared to text encoding, which are outlined in the [Improving the performance of web services in Silverlight 3 Beta](http://blogs.msdn.com/silverlightws/archive/2009/06/07/improving-the-performance-of-web-services-in-sl3-beta.aspx) post.   

#### ASP.NET and IIS7 configuration tuning  

In .NET Framework 3.5 SP1, WCF introduced a new asynchronous HTTP handler for IIS7 which allows for better scalability of WCF services by not blocking worker threads for the duration of high latency service operations. This feature is described in detail in Wenlong Dong’s blog post about [asynchronous WCF HTTP Handlers](http://blogs.msdn.com/wenlong/archive/2008/08/13/orcas-sp1-improvement-asynchronous-wcf-http-module-handler-for-iis7-for-better-server-scalability.aspx). In order to realize the full potential of asynchronous HTTP handlers, the concurrent HTTP request quota (MaxConcurrentRequestsPerCPU registry setting) also needs to be adjusted, which is described in more detail in Thomas Marquardt’s post about [threading in IIS 6.0 and IIS 7.0](http://blogs.msdn.com/tmarq/archive/2007/07/21/asp-net-thread-usage-on-iis-7-0-and-6-0.aspx).   

Registration of the asynchronous HTTP handler for WCF as well as adjustment of the quota for concurrent HTTP requests can we performed with [Wenlong Dong’s WcfAsyncWebTool](http://blogs.msdn.com/wenlong/attachment/8856601.ashx):  

{% highlight text linenos %}


WcfAsyncWebTool.exe /ia /t 20000 

{% endhighlight %}

  

For the purpose of this test, we are setting the quota at 20000 to ensure it is not going to become the limiting factor in measuring the maximum number of clients the server can accommodate. In production, setting this limit should take into account on one hand a sustainable number of clients (resulting from performance measurements of the actual scenario), as well as an acceptable working set.   

Following the setting of the quota for concurrent HTTP requests to 20000, the quotas for the thread pool worker threads and IO threads also need to be adjusted through .NET Configuration in machine.config:  

{% highlight text linenos %}


<processModel maxWorkerThreads="20000"        
                      maxIoThreads="20000"    
                      minWorkerThreads="10000"/> 

{% endhighlight %}

  

These settings ensure there is adequate supply of threads to handle the HTTP requests IIS7 will accept given the concurrent HTTP request quota, as well as for the service to send  bursts of asynchronous notifications to clients.   

#### WCF code tuning  

WCF has made it easy to author a well performing RPC service, but requirements and messaging patterns of a pub/sub service are sufficiently different from RPC to require a few performance optimizations in code.   

One consideration is that a pub/sub service often sends out multiple identical messages to several clients, in particular in the broadcast scenario. Given that a substantial portion of the cost of sending a message is serializing its content, it is worthwhile to pre-serialize a message once and then send a copy of it multiple times. In order to accomplish this, the callback contract of a pub/sub service should take a Message as a parameter as opposed to typed parameters. This allows the message to be pre-serialized and converted to a MessageBuffer using TypedMessageConverter. Then, for every notification to be sent, the MessageBuffer can be used to create a clone of the Message without incurring the serialization cost.   

Sending a large number of notifications from the server is a high latency operation. In order to optimize thread use, sending the notifications to clients in a loop should use asynchronous methods of the callback contract, or enqueue the synchronous invocations using thread poll thread. We did not measure a meaningful difference in performance between these two methods.   

The concurrency mode of the WCF service should be set to ConcurrencyMode.Multiple, and instance mode to InstanceMode.Single. This requires explicit synchronization code to be added around access to critical resources (e.g. shared data structures), but the extra effort pays off in reduced contention of concurrent requests to the service.   

All of these optimizations are demonstrated in [the reference implementation of a pub/sub WCF service using the HTTP polling duplex protocol](http://tomasz.janczuk.org/2009/07/pubsub-sample-using-http-polling-duplex.html).   

  

  

### Results  

The following server configuration was used in all measurements: 4-proc Intel Xeon 2.66GHz, 4GB RAM, Windows Server 2008 SP1 64bit with .NET Framework 3.5 SP1 and IIS7.   

Clients were run on machines other than the server. The number of machines and clients running on each machine was adjusted to achieve the point of saturation of the server.   

Message format and content was the same in all measurements. The message consisted of 20 short strings. Although we don’t have formal results as a function of a message size, ad-hoc measurements indicated the results were not affected by message sizes between 1 and 100 strings in a meaningful way.   

#### Broadcast scenario results  

The broadcast scenario measurement was based on the following script:  

1. M clients subscribed to a single topic on the server.  
2. The server started generating messages to be published to the topic every P seconds. Upon publishing of a message, M notifications were sent to M clients subscribed to the topic (one per client), which we called a “burst”.  
3. Several runs of the test were performed for increasing numbers of M, up to the point where the server was unable to finish sending all messages in a burst before the next burst was due to be sent. The largest value of M at which the server was able to send all messages of each of at least 100 consecutive bursts before the next burst became due was considered the result of the test for a given burst frequency P.  
  

Results of this measurement are shown on the chart below:  

 ![broadcast_results](http://lh5.ggpht.com/_NUp_nWDyyvI/SoUC0FWPbrI/AAAAAAAAA-M/BMNGRWxpcAM/broadcast_results_thumb%5B1%5D.png?imgmax=800)   

An example of interpreting this data is that a single server using the HTTP polling duplex protocol from Microsoft Silverlight 3 can support sending notifications to 5000 connected clients every 10 seconds.  

#### Collaboration scenario results  

The collaboration scenario measurement was based on the following script, simulating a server supporting multiple chartrooms with many participants each:   

1. T topics (chat rooms) are created on the server.  
2. N distinct clients subscribe to every one of the T topics (the total number of clients connected to the server is then N * T).  
3. One of the clients subscribed to any given topic publishes a single message to that topic. This happens simultaneously for all topics. The server broadcasts the message back to the N-1 other clients subscribed to that topic immediately after receiving the publish message.  
4. After all N-1 clients to whom notifications were sent have received them, no activity occurs in the chat room (topic) for D seconds. The time between the publish message message was sent by the publisher to a topic and received by a subscribers to that topic is captured; we call this metric a latency. Each publish event generates N-1 data points, one for every subscriber of a topic other than the publisher.  
5. Every topic (chat room) repeats the #3-#4 cycle independently (simultaneously) over a minimum of 100 iterations.  
6. The latencies gathered across all iterations, topics, and subscribers are statistically analyzed.  
  

The results for a few distinct sets of defining parameters (number of topics T, number of subscribers per topic N, delay between publications D) are presented below. For each of the variations we have measured the [mean](http://en.wikipedia.org/wiki/Mean), [median](http://en.wikipedia.org/wiki/Median), and [standard deviation](http://en.wikipedia.org/wiki/Standard_deviation) of the notification latency.   
<table>     <tr>       <td>         

Topics T       </td>        <td>         

Subscribers per topic N       </td>        <td>         

Total clients N * T       </td>        <td>         

Delay between publications D [s]       </td>        <td>         

Mean latency [ms]       </td>        <td>         

Median latency [ms]       </td>        <td>         

Stdev [ms]       </td>     </tr>      <tr>       <td>         

100       </td>        <td>         

5       </td>        <td>         

500       </td>        <td>         

1       </td>        <td>         

91       </td>        <td>         

63       </td>        <td>         

122       </td>     </tr>      <tr>       <td>         

500       </td>        <td>         

2       </td>        <td>         

1000       </td>        <td>         

15       </td>        <td>         

4       </td>        <td>         

0       </td>        <td>         

12       </td>     </tr>      <tr>       <td>         

500       </td>        <td>         

3       </td>        <td>         

1500       </td>        <td>         

15       </td>        <td>         

108       </td>        <td>         

94       </td>        <td>         

73       </td>     </tr>      <tr>       <td>         

800       </td>        <td>         

3       </td>        <td>         

2400       </td>        <td>         

15       </td>        <td>         

2743       </td>        <td>         

1328       </td>        <td>         

3195       </td>     </tr>      <tr>       <td>         

1000       </td>        <td>         

2       </td>        <td>         

2000       </td>        <td>         

5       </td>        <td>         

496       </td>        <td>         

498       </td>        <td>         

288       </td>     </tr>      <tr>       <td>         

1000       </td>        <td>         

2       </td>        <td>         

2000       </td>        <td>         

15       </td>        <td>         

6       </td>        <td>         

0       </td>        <td>         

17       </td>     </tr>      <tr>       <td>         

2000       </td>        <td>         

2       </td>        <td>         

4000       </td>        <td>         

15       </td>        <td>         

25       </td>        <td>         

0       </td>        <td>         

99       </td>     </tr>   </table>
  

Whenever the median latency shows 0ms, it indicates the latency in over 50% of data points was below the threshold of a time span we could capture.   

The data indicates a single server can support 2000 simultaneous chat rooms with 2 participants each and a 15 second delay between publications with a 25ms mean latency (0 ms median latency), which should satisfy latency requirements of most UI driven scenarios. At the same time, the data shows that the latency gets out of hand with 800 chatrooms with 3 participants each and 15 second delay between publications.   }