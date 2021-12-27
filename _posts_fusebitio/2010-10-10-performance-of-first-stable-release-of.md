---
tags: ['post']
post_og_image: 'site'
date: '2010-10-10'  
post_title: Performance of the first stable release of the Laharsub pub/sub
  service for web clients
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: performance-of-first-stable-release-of
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




### Overview  

This article discusses performance of the first stable release of the Laharsub pub\sub server for web clients ([http://laharsub.codeplex.com](http://laharsub.codeplex.com)). All measurements presented here were generated using the [Laharsub 2010.10.09 release](http://laharsub.codeplex.com/releases/view/53694).   

The goal of the open source Laharsub project is to provide a solution that makes it easy for web applications to organize internet scale message exchange using a publish/subscribe pattern. The project is an ongoing experiment with a variety of web technologies. Current focus is on AJAX (in particular jQuery) and Silverlight clients, a REST based HTTP long polling subscription protocol implemented by a .NET WCF HTTP middle tier service, and researching middle tier and back end technologies that enable scale-out to a large number of clients.   

Laharsub 2010.10.09 release provides the Laharsub service based on WCF HTTP service, an in-memory pub\sub backend implementation, and three client implementations: jQuery, Silverlight, and .NET.   

### Methodology  

The test scenario used for the purpose of performance measurements simulates a chat/collaboration message exchange pattern. The server maintains a specified number of topics. Each topic has the same number of unique clients acting both as publishers and subscribers of the topic. Each of the clients is publishing messages to the topic at specified frequency. Upon receiving a message published to a topic, the Laharsub server sends notifications containing this message to all subscribers of the topic (including the client which published the message).   

The following three configurations were tested:  

1. Variations in the number of topics, with 2 clients per topic, each publishing messages to the topic every 1 second.  
2. Variations in the number of topics, with 2 clients per topic, each publishing messages to the topic every 5 seconds.  
3. Variations in the number of topics, with 5 clients per topic, each publishing messages to the topic every 15 seconds.  
  

The key quality of service metric of every measurement is the roundtrip latency of a message, measured as the time elapsed between the moment a message was published to a topic by a client and the moment the message was received by a client subscribing to that topic. In the context of a chat scenario, this is the time required for a message typed into a messaging client to reach the other participant or participants of the chat.   

The client application simulating the chat/collaboration scenario is called LoadTestClient.exe and is included in the Laharsub release. Each run of the test lasts 30 seconds, preceded by 10 seconds of a warm-up operation intended for the system to reach a steady state. Measurements are recorded only after the warm-up period. Every message published to a topic is 10 bytes long and consists of a timestamp representing the moment the message was created on the client that publishes it. When a client subscribing to the topic receives that message, it calculates the time elapsed from its creation and records this data as a single measurement. In the course of the 30 second test run, a large number of measurements are recorded. Arithmetic average and standard deviation of the latency are calculated from all these measurements for a given variation.   

The tests have been run in the following environment:  

1. The Laharsub.exe server was running as a console application on a 64 bit Windows 2008 R2 operating system on a Intel Quad Core 2.4GHz machine with 4GB of RAM and a 100MB/s network card.  
2. There were two client machines used in every variation (except one involving a single topic, in which case a single machine was used). Each machine was running LoadTestClient.exe on a 32 bit Windows 7 Ultimate operating system on an Intel Dual Core 2.2GHz with 2GB of RAM and a 100MB/s network card.  
3. The two client machines and the server were on a local network connected with a 100MB/s switch.  
  

For each test variation testes the following metrics were gathered:  

1. Average and standard deviation of notification latency, as described above.  
2. Average memory utilization of the Laharsub.exe server (private bytes).  
3. Average network utilization of the server machine.  
  

There are numerous factors that affect latency of message delivery as well as other metrics, and it is certain most of them are going to be different in a production environment than the test environment described above. Therefore only consider the results discussed below as a general guidance, and measure performance of the Laharsub server in your particular application. In particular, it is to be expected the network environment (e.g. distance between the clients and the server) are going to be a major factor that impacts message latency. In that sense, the test environment above represents close to an ideal case for achieving low notification latency.   

### Latency  

Average latency measured for all of the variations of each of the test configurations are presented on Figure 1 below.   

 ![f1.average.latency](http://download.codeplex.com/Project/Download/FileDownload.aspx?ProjectName=laharsub&DownloadId=156325)  

It must be noted that the Laharsub server was not the bottleneck of scalability to a larger number of concurrent clients in any of the variations depicted on Figure 1. In every case the bottleneck was CPU utilization on the client machine. (Unfortunately I do not have more client machines at my disposal to try to load the network or the server). Server utilization is discussed more in the following sections.   

Apart from average notification latency, standard deviation of the latency measurements shown on Figure 2 provides more insight into the overall quality of service.   

 ![f2.stddev.latency](http://download.codeplex.com/Project/Download/FileDownload.aspx?ProjectName=laharsub&DownloadId=156327)  

It is apparent from the two graphs that the average and standard deviation of notification latencies are correlated. Figure 3 and Figure 4 provide a more graphical representation of the distribution of latency measurements for individual notifications for points identified as A and B on Figure 1, respectively.  

 ![f3.200-2-1.latency.distribution](http://download.codeplex.com/Project/Download/FileDownload.aspx?ProjectName=laharsub&DownloadId=156329)  

 ![f4.5000-5-15.latency.distribution](http://download.codeplex.com/Project/Download/FileDownload.aspx?ProjectName=laharsub&DownloadId=156331)  

One interesting observation about the notification distribution in time is the apparent arrival of several notifications at the same moment. This observation demonstrates the benefit of Laharsub protocol multiplexing several topic subscriptions on a single HTTP long poll request, which in turn causes several notification to be sent back to client on a single HTTP long poll response. These notifications appear to arrive at the same time since the start of the measurement. If the multiplexing optimization was absent, these notifications would require a separate HTTP long poll response each, which in turn would increase the average notification latency.   

### Detailed measurements  

As mentioned before, three configurations were tested:  

1. Variations in the number of topics, with 2 clients per topic, each publishing messages to the topic every 1 second.  
2. Variations in the number of topics, with 2 clients per topic, each publishing messages to the topic every 5 seconds.  
3. Variations in the number of topics, with 5 clients per topic, each publishing messages to the topic every 15 seconds.  
  

Details of average and standard deviation of the latency, as well as memory and network utilization on the server for all of the configurations above are presented on Figure 5, 6, and 7 below, respectively.   

 ![f5.2-1.summary](http://download.codeplex.com/Project/Download/FileDownload.aspx?ProjectName=laharsub&DownloadId=156333)  

 ![f6.2-5.summary](http://download.codeplex.com/Project/Download/FileDownload.aspx?ProjectName=laharsub&DownloadId=156335)  

 ![f7.5-15.summary](http://download.codeplex.com/Project/Download/FileDownload.aspx?ProjectName=laharsub&DownloadId=156337)  

One interesting observation is that network utilization (as measured on the server) in every configuration grows linearly with the number of concurrent clients. This should not be surprising, given a fixed message size and fixed publication frequency in all of the above configurations.   

The data above also seems to suggest the average and standard deviation of notification latency grows non-linearly with the number of concurrent clients, an effect that is further amplified by a more higher publication frequency.   

### Server memory  

Consumption of memory by the server process (private bytes) for the three configuration tested is shown on Figure 8.   

 ![f8.memory](http://download.codeplex.com/Project/Download/FileDownload.aspx?ProjectName=laharsub&DownloadId=156339)  

It appears that memory consumption grows linearly with the number of concurrent clients in each of the three configurations. There appear to be two factors affecting memory consumption: concurrent connections on the server, and the number of messages held in memory by the server. In the test scenario, the memory consumed by messages stored in memory is negligable, provided the 10 byte size of every message and the configured message TTL of 15 seconds. It is to be expected that message size and TTL may contribute substantially to the memory consumption in other deployments. The memory cost of supporting a specific number of concurrent clients (concurrent TCP connections) is independent of the message size.   

### Network utilization  

Network utilization as measured on the server for all three test configurations is shown on Figure 9 below.   

 ![f9.network](http://download.codeplex.com/Project/Download/FileDownload.aspx?ProjectName=laharsub&DownloadId=156341)  

It is not surprising that network utilization grows linearly with the number of messages exchanged. One aspect to be noted is that despite the relatively large overhead of the HTTP protocol over the body payload (10 bytes) in the HTTP long polling protocol that the Laharsub server is using, in none of the configurations did the network utilization of a 100MB/s interface exceed 18%. As discussed before, this is partially due to the subscription and notification multiplexing on a single HTTP long poll request.   