---
tags: ['post']
post_og_image: 'site'
date: '2010-06-24'  
post_title: Duplex communication with WCF on Silverlight TV
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: duplex-communication-with-wcf-on
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




Silverlight TV episode about [Duplex communication with WCF in Silverlight](http://channel9.msdn.com/shows/SilverlightTV/Duplex-Communication-with-WCF-in-Silverlight-4-Silverlight-TV-34/) is beginning to air today. In this episode I am showing how to implement duplex communication with WCF using HTTP polling duplex protocol, as well as discuss aspects of the net.tcp protocol. If you want to find out more about the technologies I used to build the Silverlight-based chat application in this short video, you may want to check out the following resources:  

* [Pub/sub sample using HTTP polling duplex channel in Microsoft Silverlight](http://tomasz.janczuk.org/2009/07/pubsub-sample-using-http-polling-duplex.html) contains the source code of a more elaborate version of the chat application, and walks through key design points.  
* [AJAX client for HTTP polling duplex WCF channel](http://tomasz.janczuk.org/2009/08/ajax-client-for-http-polling-duplex-wcf.html) explains how a JavaScript browser client can consume the HTTP polling duplex protocol from Silverlight. This post builds on the previous one to show how AJAX and Silverlight based clients can communicate with each other using a pub\sub WCF backend exposed over the HTTP polling duplex binding from Silverlight. It comes with a JavaScript library implementing the client side of the protocol.  
* [Scale-out of HTTP polling duplex WCF service](http://tomasz.janczuk.org/2009/09/scale-out-of-silverlight-http-polling.html) discusses the challenges and approaches to scaling out a WCF service exposed over the HTTP polling duplex binding to a large number of clients.  
* [Pub/sub sample using the net.tcp channel in Microsoft Silverlight](http://tomasz.janczuk.org/2009/11/pubsub-sample-with-wcf-nettcp-protocol.html) builds up on the tired chat example to show how a Silverlight client can seamlessly switch between HTTP polling duplex and net.tcp protocol added in Silverlight 4, and explains some of the trade-offs between these protocols.  
* [WCF net.tcp protocol in Silverlight 4](http://tomasz.janczuk.org/2009/11/wcf-nettcp-protocol-in-silverlight-4.html) provides more background information about the protocol in the context of Silverlight.  
* [Comparison of net.tcp and HTTP polling duplex performance](http://tomasz.janczuk.org/2010/03/comparison-of-http-polling-duplex-and.html) provides an insight into and helps set expectations related to relative performance of the two protocols in various circumstances.  
* The [Silverlight Web Services Team](http://blogs.msdn.com/b/silverlightws/) blog contains a wealth of information about WCF and communication in Silverlight in general.  
  

I hope you enjoy [the video](http://channel9.msdn.com/shows/SilverlightTV/Duplex-Communication-with-WCF-in-Silverlight-4-Silverlight-TV-34/). Make sure to [check out other Silverlight TV episodes](http://channel9.msdn.com/shows/SilverlightTV/) as well.   