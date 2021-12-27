---
tags: ['post']
post_og_image: 'site'
date: '2009-07-09'  
post_title: Adoption of WCF technologies in Silverlight 2 applications on the web
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: adoption-of-wcf-technologies-in
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




Microsoft Silverlight 2 is the first version of the .NET client development platform which contains Windows Communication Foundation (WCF) components. The Silverlight platform in this version is primarily targeting rich internet application development. Such applications execute in a security sandbox that limits the choices for data exchange with the outside world. For example, a Silverlight application does not have free access to the file system except for an isolated local storage. Any Silverlight application that requires external data must use a communication mechanism to obtain it from the web and a serialization mechanism to parse and manipulate the data. Silverlight 2 offers several communication and data serialization technologies that facilitate these scenarios.   

The choice of communication and serialization components in Silverlight 2 was based on developer community data and the target scenarios for the platform. Some time after the platform has shipped in October 2008 I decided to measure the level of adoption of the various technologies we added to Silverlight 2. We intended to use this data to validate our original assumptions as well as inform future releases.   

The methodology of this measurement was based on finding a statistically significant sample of Silverlight 2 applications deployed on the web and statically analyzing these applications to identify the platform features they use. I was interested in several communication technologies:  

* WebClient  
* HttpWebRequest  
* ClientBase (WCF proxy)  
* ChannelFactory (direct use of WCF)  
* Sockets  
* Polling duplex  
* Astoria  
  

I was also looking for several serialization technologies:  

* DataContractSerializer  
* DataContractJsonSerializer  
* XmlSerializer  
* JsonValue  
* Syndication  
* XElement  
* XmlReader  
  

Lastly, I was interested in the backend technology of services the clients were communicating with (ASMX, WCF, or other), as well as whether these calls were same-domain or cross-domain.   

In February 2009 (about 5 months after Silverlight 2 released) I used a custom web crawl engine to index over 40,000 web pages, which yielded 772 Silverlight 2 applications across 335 web sites. Then I downloaded and disassembled the managed application code from the XAP into IL to statically analyze it for sentinel constructs of the technologies in question. I discuss the results below.   

The high level findings are as follows:  

* 53% of applications used one or more of the communication technologies  
* 47% of applications used one or more of the serialization technologies (counting only direct use by the application code; this number does not include indirect use via other components, e.g. a WCF proxy)  
* 41% of applications used neither communication or serialization technologies  
  

The fact that over half of all Silverlight applications used at least one communication technology emphasizes the importance of the communication component in a sandboxed web application.   

The table below contains the breakdown of the adoption of individual communication technologies across applications that use at least one of the communication technologies (i.e. across the 53% of Silverlight 2 applications I found through the web crawl).   
<table>     <tr>       <td>Rank</td>        <td>Adoption</td>        <td>Technology</td>     </tr>      <tr>       <td>1</td>        <td>63.6%</td>        <td>WebClient</td>     </tr>      <tr>       <td>2</td>        <td>44.9%</td>        <td>ClientBase (WCF proxy)</td>     </tr>      <tr>       <td>3</td>        <td>18.0%</td>        <td>HttpWebRequest</td>     </tr>      <tr>       <td>4</td>        <td>2.2%</td>        <td>Socket</td>     </tr>      <tr>       <td>5</td>        <td>1.7%</td>        <td>Astoria</td>     </tr>      <tr>       <td>6</td>        <td>0.7%</td>        <td>ChannelFactory (WCF)</td>     </tr>      <tr>       <td>7</td>        <td>0.2%</td>        <td>Polling duplex</td>     </tr>   </table>
  

Similarly, the table below contains the serialization technology adoption breakdown across application that use at least one of the serialization technologies (i.e. across the 47% of Silverlight 2 applications I found through the web crawl). The number represents only direct use of these serialization mechanisms by the application code. In particular, it excludes indirect use by web service proxies like ClientBase (WCF) the application may be using.   
<table>     <tr>       <td>Rank</td>        <td>Adoption</td>        <td>Technology</td>     </tr>      <tr>       <td>1</td>        <td>70.3%</td>        <td>XElement</td>     </tr>      <tr>       <td>2</td>        <td>43.5%</td>        <td>XmlReader</td>     </tr>      <tr>       <td>3</td>        <td>8.8%</td>        <td>XmlSerializer</td>     </tr>      <tr>       <td>4</td>        <td>6.9%</td>        <td>DataContractJsonSerializer</td>     </tr>      <tr>       <td>5</td>        <td>3.2%</td>        <td>DataContractSerializer</td>     </tr>      <tr>       <td>6</td>        <td>1.9%</td>        <td>JsonValue</td>     </tr>      <tr>       <td>7</td>        <td>1.6%</td>        <td>Syndication</td>     </tr>   </table>
  

WebClient is the mostly adopted communication technology, followed by the WCF proxy (ClientBase). WebClient is mainly used to download entire files or loosely structured XML data, which is later processed using XElement as a serializer of choice. WCF is primarily used as a mechanism to access strongly typed data. One aspect to note is that WCF proxy internally uses DataContractSerializer (another WCF technology) as a default serialization engine (something WebClient is not doing with XElement), which means that DataContractSerializer has a total adoption of around 25.3% across all surveyed Silverlight 2 applications (47% * 3.2% + 53% * 44.9%; this assumes no overlap between applications using DataContractSerializer directly and ClientBase at the same time, which is an approximation).   

Analysis of the backend technology of the services that Silverlight 2 clients are communicating with indicates 52% of them are based on WCF technology, 42% are based on ASMX, and the technology of the remaining 6% could not be determined. Furthermore, analysis of the URLs of the web service calls indicates 68% of calls is cross-domain while only 32% are same-domain. The large share of cross-domain calls is an interesting finding in itself. It also seems to support a hypothesis that the (still) large share of ASMX as the backend technology has to do with the existence of legacy services.  

Overall the data from this research informed several decisions related to investments in subsequent releases of Silverlight, but this is another story. As a matter of fact Silverlight 3 is coming out within hours of this writing, check out [http://www.microsoft.com/silverlight/](http://www.microsoft.com/silverlight/) for an announcement.   