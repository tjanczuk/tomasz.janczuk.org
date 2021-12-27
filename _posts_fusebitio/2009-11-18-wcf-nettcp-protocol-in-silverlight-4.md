---
tags: ['post']
post_og_image: 'site'
date: '2009-11-18'  
post_title: WCF net.tcp protocol in Silverlight 4
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: wcf-net.tcp-protocol-in-silverlight-4
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




Microsoft Silverlight 4 Beta unveiled at PDC 2009 enables Silverlight clients to communicate with a Windows Communication Foundation (WCF) service using the net.tcp protocol. The key benefits of the net.tcp protocol are:  

* support for duplex communication with a simple to use programming model,  
* excellent performance,  
* good integration with the Add Service Reference feature of Visual Studio,  
* improved library factoring to help optimize the Silverlight application (XAP) size.  
  

Practical use of the protocol is constrained to intranet environments where firewall configuration is controlled, since the net.tcp protocol is subject to the same TCP port limitations as any Silverlight application (only ports 4502-4534 can be accessed). In addition, the net.tcp protocol in Silverlight 4 does not support transport level security.   

### Duplex communication  

WCF net.tcp protocol in Silverlight 4 addresses the same duplex communication scenario as the HTTP polling duplex protocol from Silverlight 2 and 3, while providing a major performance improvement and maintaining the same, simple to use event based programming model:  

```
DuplexServiceClient client = new DuplexServiceClient("NetTcpBinding_DuplexService");        
client.ResponseReceived += new EventHandler<ResponseReceivedEventArgs>(client_ResponseReceived); 

```
  

Silverlight 3 applications that need to receive asynchronous data notifications from the server (the flagship example is stock quote updates) could choose between two technologies: WCF’s HTTP polling duplex protocol or direct use of System.Net.Socket. Between these two choices, WCF offers a much simpler to use, strongly typed programming model based on events and callback contracts, as well as a firewall friendly protocol. I have published [an example of using the HTTP polling duplex protocol to implement a pub/sub application](http://tomasz.janczuk.org/2009/07/pubsub-sample-using-http-polling-duplex.html) before.   

With the WCF net.tcp support added in Silverlight 4, client applications can continue to benefit from the programming model usability while gaining a major performance benefit over HTTP polling duplex protocol. Migration of client applications already utilizing HTTP polling duplex protocol to use net.tcp should require minimal changes in the application code. In practice, only the choice of the binding when creating a service proxy is affected. Similarly, a duplex WCF service already exposed over HTTP polling duplex endpoint will only require a new endpoint based on the NetTcpBinding from .NET Framework. This change can typically be done in configuration without modifying the service code.   

### Performance  

Performance of net.tcp protocol compared to HTTP polling duplex is the key reason to consider using the WCF net.tcp protocol in duplex communication scenarios. Most of the performance benefits of net.tcp protocol also apply to request/response messaging patterns. There are three major performance gains net.tcp offers compared to HTTP polling duplex:  

* increase in throughput (decrease in latency) of data notifications a client application can receive from the server,  
* increase in the maximum number of connected client the server can support concurrently,  
* bandwidth reduction.  
  

Let’s first inspect the throughput. I have created a casual benchmark application (code of which is available [here](http://janczuk.org/code/samples/nettcpperf.zip)) in which the server is sending a predefined number of messages to a client (each message contains 5 strings), and the client measures the time it takes to receive them all. I measured and compared two implementation variants: in one the client receives messages from the server on the UI thread of the application, and in the other the client uses a worker thread instead. The chart below shows the result of the benchmark comparing the HTTP polling duplex protocol to net.tcp (client was running on a Dell Latitude E6400 laptop):  

 ![Throughput of net.tcp and HTTP polling duplex protocols in Silvrelight 4](http://lh3.ggpht.com/_NUp_nWDyyvI/SwRHPBai05I/AAAAAAAABCs/XWAdny5jy5M/NetTcpAndHttpPollingDuplexThroughput%5B1%5D.png?imgmax=800)   

Note the throughput scale is **logarithmic**. There are a few takeaways here:   

* Throughput of net.tcp on a worker thread of a Silverlight 4 application is **87,000% greater (that is 870 times faster)** than HTTP polling duplex. Net.tcp protocol is using multiple worker threads to receive and dispatch messages from the server. The HTTP polling duplex protocol builds on top of the HTTP stack which in Silverlight 4 uses a single UI thread to receive  messages (even if the request has originated on a worker thread).  
* Throughput of net.tcp on a UI thread is 550% greater (that is 5.5x faster) than HTTP polling duplex. Net.tcp protocol streams consecutive messages to the client over an active TCP connection which does not involve per-message network roundtrip (HTTP request/response) required by the Silverlight 3 implementation of HTTP polling duplex.  
* Throughput of net.tcp on a worker thread is **16,330% greater (that is 163 times faster)** than on the UI thread. Messages net.tcp received and dispatched on multiple threads are only processed sequentially on a single UI thread if the original request was made on the UI thread. This [performance gain of the worker thread over the UI thread is a phenomenon I described in the context of the HTTP protocol](http://tomasz.janczuk.org/2009/08/improving-performance-of-concurrent-wcf.html) before.  
  

Let’s now have look at server scalability. There are two interesting aspects: how many concurrent clients can a single server maintain, and how does the protocol scale out. I have written an article about [scalability of a single server HTTP polling duplex deployment](http://tomasz.janczuk.org/2009/08/performance-of-http-polling-duplex.html) before. I have also [suggested ways the HTTP polling duplex protocol can scale out](http://tomasz.janczuk.org/2009/09/scale-out-of-silverlight-http-polling.html) in a separate post. The chart below shows two data points comparing performance of the net.tcp and HTTP polling duplex protocol on a 3.5 GHz dual core machine with 2GB RAM (not a poster child of a server machine, but something we had handy) in a broadcast scenario (server sends identical messages to a large number of clients at fixed frequency):   

 ![Scalability of the net.tcp and HTTP polling duplex protocol in Silberlight 4](http://lh5.ggpht.com/_NUp_nWDyyvI/SwRHQ9mdcFI/AAAAAAAABC0/GD_at8A1pWw/NetTcpAndHttpPollingDuplexScalabilit%5B2%5D.png?imgmax=800)   

The data shows the same hardware can support more concurrent clients with the net.tcp protocol than HTTP polling duplex. At the same time, however, scale-out of the net.tcp protocol requires backend affinity for the duration of the TCP connection, while scale out of the HTTP polling duplex protocol can be accomplished without similar guarantee at the HTTP layer (as I described [here](http://tomasz.janczuk.org/2009/09/scale-out-of-silverlight-http-polling.html)).   

Last aspect of performance to consider is network bandwidth. I don’t have concrete data to show here, but the size of a single HTTP request/response per message of the HTTP polling duplex protocol is clearly a loosing proposition to the .NET Framing Protocol used for streaming multiple messages over TCP in the net.tcp protocol.   

### Visual Studio integration  

Net.tcp protocol is well integrated with the Add Service Reference feature in Visual Studio 10. A WCF service proxy to a WCF service exposed using the net.tcp binding can be easily generated for a Silverlight 4 client application. Net.tcp protocol comes with Silverlight configuration support (ServiceReferences.ClientConfig), so the details of the binding, including service address, can be changed without recompiling of the application:  

>
> <configuration>
>
> <system.serviceModel>
>
> <bindings>
>
> <customBinding>
>
> <binding name="NetTcpBinding_DuplexService">
>
> <binaryMessageEncoding />
>
> <tcpTransport maxReceivedMessageSize="2147483647" maxBufferSize="2147483647" />
>
> </binding>
>
> </customBinding>
>
> </bindings>
>
> <client>
>
> <endpoint address="net.tcp://localhost:4502/tcpperf/DuplexService.svc"
>
> binding="customBinding" bindingConfiguration="NetTcpBinding_DuplexService"
>
> contract="Proxy.DuplexService" name="**NetTcpBinding_DuplexService**" />
>
> </client>
>
> </system.serviceModel>
>
> </configuration>  

Proxy can be instantiated in code by referring to the named endpoint in Silverlight configuration:  

```
DuplexServiceClient client = new DuplexServiceClient("NetTcpBinding_DuplexService");  

```
  

### Library factoring  

WCF library layering in Silverlight 4 has been updated to allow application (XAP) size to be optimized given the protocols that are actually used:   

 ![Factoring of WCF libraries in Silverlight 4](http://lh3.ggpht.com/_NUp_nWDyyvI/SwRHRRLVL1I/AAAAAAAABC8/GjpbxSNCjpU/ServiceModelSilverlightLayering_thum.png?imgmax=800)   

Applications that utilize only request/response exchange patterns using WCF have all the required components available in Silverlight core. Applications that require duplex message exchange patterns must include the System.ServiceModel.Extensions.dll in the XAP, as well as one or both of System.ServiceModel.NetTcp.dll and System.ServiceModel.PollingDuplex.dll, depending on the protocol the application is using. System.ServiceModel.Extensions.dll contains functionality common for all duplex communication scenarios. System.ServiceModel.NetTcp.dll supports the WCF net.tcp protocol, and System.ServiceModel.PollingDuplex.dll the HTTP polling duplex protocol.   

When a proxy is created in a Silverlight 4 project using the Add Service Reference feature of Visual Studio 10, appropriate library references will be added to the project depending on the actual protocols the WCF service exposes.   

### Limitations  

  

The WCF net.tcp protocol in the Silverlight environment has limitations that make it a better fit for some applications than others:   

* The protocol builds on top of Silverlight’s System.Net.Socket implementation and as such is subject to the same [network security restrictions](http://msdn.microsoft.com/en-us/library/cc645032(VS.95).aspx). In particular, a Silverlight application can only use TCP ports in the range 4502-4534. In practice it means the protocol is intended for use on intranets (as opposed to the public internet) as it requires environments where firewall configuration can be controlled.  
* Servers hosting WCF net.tcp services intended for consumption from Silverlight must also expose a socket policy to allow TCP connections from Silverlight clients. I have created [an online project template for Visual Studio 10 to facilitate hosting of the TCP socket policy](http://visualstudiogallery.msdn.microsoft.com/en-us/c4534af5-e864-42c2-b351-094593864e78).  
* Net.tcp protocol in Silverlight does not support transport level security (i.e. it does not offer SSL protection). As such its use is limited to applications where security is not a requirement or the desired level of security is inherent in the environment the application is running in.  
  

### Summary  

WCF net.tcp protocol in Silverlight 4 is a great fit for intranet applications that require the utmost in communication performance. The protocol supports the simple to use, event-based WCF duplex programming model. Adding strongly typed service proxy to a Silverlight project is made easy with integration with the Visual Studio 10 Add Service Reference feature. Communication and security limitations make the protocol a better fit for controlled intranet environments than the public internet. A good protocol alternative for a duplex communication for the public internet is the [HTTP polling duplex](http://tomasz.janczuk.org/2009/07/pubsub-sample-using-http-polling-duplex.html) already present in Silverlight since version 2.   }