---
tags: ['post']
post_og_image: 'site'
date: '2010-10-10'  
post_title: First stable release of the Laharsub pub/sub service for web clients
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: first-stable-release-of-laharsub-pubsub
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




After extensive stress and performance testing and improvements, here is [the first stable release of Laharsub](http://laharsub.codeplex.com/releases/view/53694). Looking forward to your comments.  

 ![f1.average.latency.small](http://download.codeplex.com/Project/Download/FileDownload.aspx?ProjectName=laharsub&DownloadId=156344)  

Make sure to check out [the article about Laharsub performance under load](http://laharsub.codeplex.com/wikipage?title=Performance) to see how Laharsub is doing and help set your expectations.  

The Laharsub 2010.10.09 release contains:  

1. Extensive stress testing and improvements of stability and reliability of the server under load.  
2. [Performance measurements](http://laharsub.codeplex.com/wikipage?title=Performance) under load.  
3. A new test client to perform your own stress testing.  
 Functionally there are no changes compared to the previous release:   

1. Laharsub pub\sub server based on .NET 4.0 WCF HTTP service that implements the REST publish/subscrive APIs for creating topics, publishing to a topic, and subscribing to topics. The implementation uses HTTP long polling protocol with subscription multiplexing for improved performance.  
2. .NET 4.0 client.  
3. Silverlight 4 client.  
4. jQuery client.  
5. Samples  
6. Unit tests  
  