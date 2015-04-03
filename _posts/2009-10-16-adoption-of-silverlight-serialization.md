---
layout: post
title: Adoption of Silverlight serialization and communication technologies in web
  applications – October 2009
date: '2009-10-16T13:35:00.001-07:00'
author: Tomasz Janczuk
tags: 
modified_time: '2009-10-16T15:24:53.101-07:00'
thumbnail: http://lh3.ggpht.com/_NUp_nWDyyvI/StjY5kFcgJI/AAAAAAAABA0/fZLr0cH2zyc/s72-c/SilverlightAdoptionOverview_thumb%5B4%5D.png?imgmax=800
blogger_id: tag:blogger.com,1999:blog-2987032012497124857.post-3616452456394040779
blogger_orig_url: http://tomasz.janczuk.org/2009/10/adoption-of-silverlight-serialization.html
---


It has been 8 months since [my previous research of the adoption of communication and serialization technologies in Silverlight applications](http://tomasz.janczuk.org/2009/07/adoption-of-wcf-technologies-in.html) deployed on the web, and a few months since Silverlight 3 shipped in July 2009. In this article I am describing the findings of a follow up research and comparing the results to the research done back in February. In addition, I expanded the report to include new interesting factors of Silverlight deployments: hosting environments and statistical differences between Silverilght 2 and 3 applications.   


### Methodology 

The research methodology remains largely unchanged from February:   


1. Build a database of Silverlight applications deployed on the web using a custom web crawler.  
2. Unpack XAPs and disassemble managed libraries to IL.  
3. Analyze IL for sentinel constructs.  
4. Inspect ServiceReferences.ClientConfig.  
5. Inspect HTTP response headers.  
6. Inspect AppManifext.xaml.  


Using the above mechanisms, I gathered the following data for each Silverlight application:  


1. Communication technologies in use: WebClient, HttpWebRequest, WCF (ClientBase), WCF (ChannelFactory), Sockets, WCF (Polling Duplex), Astoria.  
2. Serialization technologies in use: DataContractSerializer, DataContractJsonSerializer, XmlSerializer, JsonValue, Syndication, XElement, XmlReader.  
3. Technology of the backend web service, if any: ASMX, WCF, or other.  
4. Hosting environment (web server, OS).  
For the purpose of this research, I have collected and analyzed 389 Silverlight applications deployed across 238 sites on the web. Statistical results are presented below.  


### Overview

The charts below provide the break down of the Silverlight version used across the researched applications and sites hosting these applications:  
 ![Silverlight version adoption across applications and sites](http://lh3.ggpht.com/_NUp_nWDyyvI/StjY5kFcgJI/AAAAAAAABA0/fZLr0cH2zyc/SilverlightAdoptionOverview_thumb%5B4%5D.png?imgmax=800)  

Three months after Silverlight 3 shipped, the share of Silverlight 3 applications in all researched Silverlight applications remains at relatively healthy 21%.   

The break down below shows how many Silverlight applications are using at least one of the communication and serialization technologies under consideration in this research:  
 ![Adoption of at least one serialization and communication technology in Silverlight applications](http://lh5.ggpht.com/_NUp_nWDyyvI/StjY8N5k15I/AAAAAAAABA8/7ris61JeibQ/SilverlightAdoptionOverview2_thumb%5B3%5D.png?imgmax=800)   

The share of Silverlight applications utilizing at least one communication or serialization technology has increased around 10% between February and October. It appears the applications have become more sophisticated in this period.   


### Communication technologies

Breakdown of the adoption of the communication technologies under consideration is shown on the chart below:  
 ![Adoption of communication technologies in Silverlight applications](http://lh4.ggpht.com/_NUp_nWDyyvI/StjY9jW1CPI/AAAAAAAABBE/H4k51ec3uMc/SilverlightCommunicationAdoption_thumb%5B3%5D.png?imgmax=800)   

Several interesting observations can be made based on this data:  


* All new (Silverlight 3) applications show increased use of both WebClient and WCF (ClientBase and configuration) compared to February.  
* WCF adoption has reached 50% (of the 59.3% of all Silverlight 3 applications that use at least one communication technology). This means WCF is used in close to 30% of all new Silverlight 3 applications, which may indicate substantial demand for exchanging structured data with a backend.  
* For new Silverlight 3 applications, WCF as the backend web service technology has taken share from ASMX compared to February.  
* Direct use of HttpWebRequest has decreased.  


### Serialization technologies  


Breakdown of the adoption of the serialization technologies under consideration is shown on the chart below:  
 ![Adoption of serialization technologies in Silverlight applications](http://lh3.ggpht.com/_NUp_nWDyyvI/StjY-a2V0xI/AAAAAAAABBM/wYk5sJH05XU/SilverlightSerializationAdoption_thumb%5B3%5D.png?imgmax=800)   

A few observations related to this data are interesting:  


* All new Silverlight 3 applications show substantially decreased use of XElement (~15% drop). At the same time XmlReader has shown comparable increase in adoption.  
* The demand for JSON format serialization has increased, driving the adoption of DataContractJsonSerializer and JsonValue.  
* Direct use of DataContractSerializer remains very low (however, DataContractSerializer is still used indirectly through all WCF proxies in close to 30% of new Silverlight 3 applications).  


### Backend technology

The charts below show the backend technology used by applications utilizing one of the client communication technologies considered in this research:  
 ![Backend technology of Silverlight communication applications](http://lh5.ggpht.com/_NUp_nWDyyvI/StjZAA-Ek6I/AAAAAAAABBU/QWrIZxtDUrY/BackendTechnology_thumb%5B3%5D.png?imgmax=800)  

The determination of the type of the backend is based on the URL of the resource (presence of *.asmx for ASMX and *.svc for WCF). Given that, “unknown” includes static files or web services based on a technology other than .NET. A few observations from this data:  


* Silverlight applications that use WCF client have very high affinity to .NET services on the backend ; 95% of such applications are communicating with ASMX or WCF service on the backend.  
* Applications using WebClient communicate mostly with “unknown” backend types. This makes sense given that WebClient is a good fit for fetching static files from the backend as well as communicating with ad-hoc, non-.NET web services.  


### Hosting  


The charts below describe the hosting environment of all sites hosting one or more of the Silverlight applications I researched:  
 ![Hosting platform of Silverlight applications](http://lh6.ggpht.com/_NUp_nWDyyvI/StjZBKzJfjI/AAAAAAAABBc/KPCCuKDb9jg/SilverlightHosting_thumb%5B3%5D.png?imgmax=800)   

A few notable observations based on this data:  


* Large portion (around 20%) of sites hosting Silverlight are using Linux and Apache (or other web servers like Nginx).  
* Silverlight applications using WCF client have a much higher affinity to a Windows platform on the backend (95%) compared to an average Silverlight application (78%), as well as Silverlight applications using non-WCF communication technologies (83%).  
