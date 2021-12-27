---
tags: ['post']
post_og_image: 'site'
date: '2010-03-27'  
post_title: Comparison of HTTP polling duplex and net.tcp performance in Silverlight 4 RC
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: comparison-of-http-polling-duplex-and
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




Silverlight 4 RC that shipped recently at MIX 2010 supports a new mode of the HTTP polling duplex protocol with greatly improved performance compared to the version in Silverlight 3. This post compares the performance of the three mechanisms for asynchronous data push from the server to the client available in Silverlight 4 RC: the net.tcp protocol, and the two modes of the HTTP polling duplex protocol (SingleMessagePerPoll and MultipleMessagesPerPoll).   

The net.tcp protocol has been added in Silverlight 4 Beta2. It enables duplex communication with a WCF service exposed using the net.tcp binding. You can read more about the net.tcp protocol in the previous two articles: [an introductory post about the net.tcp protocol in Silverlight 4](http://tomasz.janczuk.org/2009/11/wcf-nettcp-protocol-in-silverlight-4.html), and  [a pub/sub sample using net.tcp protocol](http://tomasz.janczuk.org/2009/11/pubsub-sample-with-wcf-nettcp-protocol.html).   

The HTTP polling duplex protocol has been supported since Silverlight 2. At the high level the protocol uses [HTTP long polling mechanism](http://en.wikipedia.org/wiki/Push_technology) to enable the server to send messages to the client asynchronously. The only mode supported by the protocol until Silverlight 4 is described in some depth in my article on the [scale-out of the HTTP polling duplex protocol](http://tomasz.janczuk.org/2009/09/scale-out-of-silverlight-http-polling.html). The distinguishing characteristic of this mode is that the server can only send one message back to the client per each HTTP long poll. This mode, now called SingleMessagePerPoll, continues to be supported in Silverlight 4 for backwards compatibility.   

The new HTTP polling duplex mode added in Silverlight 4 RC enables the server to send multiple messages back to the client using a single HTTP long poll response. In cases where the number of messages the server needs to send to the client is large, this mode can provide dramatic improvements in communication performance compared to the SingleMessagePerPoll mode.  The new mode, called MultipleMessagesPerPoll, uses .NET Framing protocol to frame multiple logical messages on a single HTTP response. The resulting binary octet stream is sent using HTTP response chunking enabled in Silverlight 4 which may enable the client to receive some of the messages before the server is done sending them. In addition, WCF’s binary session encoding is used to encode all messages sent over the single poll response which reduces the bandwidth consumption (ad-hoc measurements indicate about 50% reduction compared to sending the same set of test messages using text encoding).    

### Performance benchmark  

To compare *relative* performance of the net.tcp protocol and the two modes of the HTTP polling duplex, I have created a small benchmark application. The application does not aspire to simulate real-world scenario: there is a single Silverlight client and a single WCF backend server, with the server sending a large number of messages to the client. Given the structure of test, the results of the benchmark should not be used for anything other than setting expectations about the relative performance of the three protocols. In particular, real-world performance will be affected by several factors including the number of concurrent clients, server and client hardware, and network environment. Although I am showing absolute throughput numbers below, they should be viewed as the results of an “optimistic case” on the given hardware, since a situation where there is only one client connected to a server is highly unrealistic. It is best to view these results as providing an idea about *relative* performance of the protocols.   

A few words about the configuration of the benchmark: server and client were running on a single Intel dual-core 2.26GHz box with 4GB RAM, Windows 7 Ultimate, IIS7, and .NET Framework 3.5 SP1. The client was Silverlight 4 RC running in IE 8. After the client sent a single request message to server, the server responded by sending a requested number of messages back to the client using the selected protocol. The client measured the time it took to receive all the messages, and calculated the resulting throughput. Each measurement was taken 10 times and the average was calculated. Moreover, two variations of the test were conducted: in one the client was sending and receiving messages on the UI thread of the Silverlight application, in the other a worker thread was used. The code of the benchmark application can be downloaded from [here](http://janczuk.org/code/samples/nettcpperf.zip).   

Results are presented on the graph below. Please note the scale is logarithmic.   

 ![Relative performance of net.tcp and HTTP polling duplex protocols in Silverlight 4 RC](http://lh5.ggpht.com/_NUp_nWDyyvI/S66W5M7y7YI/AAAAAAAABTI/s0OS8hRPcUk/image_thumb2.png?imgmax=800)  

### Conclusions  

First and foremost, the performance increase of the new MultipleMessagesPerPoll mode of the HTTP polling duplex protocol, compared to the SingleMessagePerPoll mode supported since Silverlight 2, is a whooping **91,000%** (that is 910 times faster) on a worker thread. In fact, using HTTP response chunking to send multiple messages back to the client using a single HTTP response allows the HTTP polling duplex protocol to achieve **88%** of the net.tcp performance on a worker thread. This is a great result considering net.tcp is the WCF protocol that offers the best throughput, and the 12% performance loss compared to net.tcp is a price well worth paying for the lack of [restrictions associated with using net.tcp in the Silverlight applications](http://tomasz.janczuk.org/2009/11/wcf-nettcp-protocol-in-silverlight-4.html).   

All but one variations of the test where the client initiated communication from the worker thread are substantially faster than corresponding UI thread variations. Silverlight application is running a single UI thread at a time, while there may be several worker threads created. So in case of the UI thread variations the communication bottleneck was clearly related to the necessity to synchronize response processing on a single thread on the client side. One surprising exception to this rule is the SingleMessagePerPoll mode, which shows the same performance on the worker thread and the UI thread. This is related to a combination of two factors. First, in the SingleMessagePerPoll mode, every message from the server to the client requires a new HTTP long poll from the client. Second, the HTTP implementation in Silverlight 4 synchronizes low level operations on the UI thread even if the request originated on a worker thread (which is a known limitation that will be addressed in future versions). MultipleMessagesPerPoll mode does not suffer from this constraint, since sending multiple messages from the server to the client requires only a single HTTP request.   

Worth calling out is also the performance benefit of binary session encoding (which is the default) compared to text encoding in the MultipleMessagesPerPoll mode on a worker thread. Binary encoding offers **138%** of the throughput of text encoding. This is related to reduced processing cost of binary XML compared to text XML; not to mention the reduction of network bandwidth (~50% of text encoding).   