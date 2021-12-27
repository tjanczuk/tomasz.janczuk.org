---
tags: ['post']
post_og_image: 'site'
date: '2009-11-18'  
post_title: Pub/sub sample with WCF net.tcp protocol in Silverlight 4
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: pub/sub-sample-with-wcf-net.tcp-protocol-in-silverlight-4
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




**NOTE** Many people reading this post are really looking for a publish\subscribe solution for Silverlight to implement a web chat or a stock quote application for the browser, a multiplayer online game, or some form of a web based collaboration tool, among others. If that fits your profile, you may want to check out http://laharsub.codeplex.com. Laharsub is an open source pub\sub message server for web clients, including Silverlight. It aspires to address the needs of people like you, who are interested in the subject matter of this post as well as willing to contribute your scenarios and requirements by leaving a comment. Enjoy!  

Microsoft Silverlight 4 Beta unveiled at PDC 2009 adds support for WCF net.tcp protocol. The protocol enables duplex communication (sending asynchronous messages from the server to the client), and greatly improves performance compared to HTTP polling duplex protocol in Silverlight 2 and 3. This article demonstrates the use of the net.tcp protocol in the context of a pub/sub Silverlight 4 application. I am starting with [the pub/sub application I used to demonstrate duplex capabilities of the HTTP polling duplex protocol before](http://tomasz.janczuk.org/2009/07/pubsub-sample-using-http-polling-duplex.html), and explaining the steps necessary to extend it to leverage the net.tcp protocol.   

[The full Visual Studio 10 Beta 2 solution of this Silverlight 4 sample is available for download](http://janczuk.org/code/samples/pubsubsamplenettcp.zip).   

Once you get the sample running, you will be able to establish asynchronous communication based on a pub/sub architecture with a single WCF service in the backend and three kinds of browser-based clients: Silverlight client using the WCF net.tcp protocol, Silverlight client using the HTTP polling duplex protocol, and AJAX client using the HTTP polling duplex protocol.   

For more in-depth look at net.tcp, check out the most recent post on [WCF net.tcp in Silverlight 4](http://tomasz.janczuk.org/2009/11/wcf-nettcp-protocol-in-silverlight-4.html).  

### Overview  

At high level, these were the most interesting steps necessary to convert [the HTTP polling duplex sample I described before](http://tomasz.janczuk.org/2009/07/pubsub-sample-using-http-polling-duplex.html) to support WCF net.tcp protocol added in Silverlight 4:  

1. Get the necessary tools: [Visual Studio 10 Beta2](http://www.microsoft.com/visualstudio/en-us/try/default.mspx#download) and [Silverlight 4 Tools for Visual Studio 10 Beta2](http://www.microsoft.com/downloads/details.aspx?FamilyID=9fa8afe9-cad6-4090-a7f6-7d9cdc560e2d&displaylang=en).  
2. Add support for net.tcp to the WCF duplex service already exposed over the HTTP polling duplex protocol.  
3. Publish the service to IIS7.  
4. Enable net.tcp activation in IIS7.  
5. Enable net.tcp protocol in IIS7.  
6. Allow Silverlight applications to communicate over TCP.  
7. Update the WCF service proxy in the Silverlight 4 application.  
8. Run and enjoy the sample.  
  

I will describe steps 2-8 in more detail next.   

### Adding net.tcp support to a WCF pub/sub service  

The [pub/sub sample for HTTP polling duplex](http://tomasz.janczuk.org/2009/07/pubsub-sample-using-http-polling-duplex.html) came with a WCF duplex service implementing the pub/sub logic. Enabling net.tcp protocol support for this service does not require any changes in the application code, and can be accomplished by adding a new endpoint in the web.config file (additions highlighted in bold):  

```
    <system.serviceModel>        
        <extensions>         
        <bindings>         
            <pollingDuplexBinding>         
                <binding name="PubSub" useTextEncoding="true"/>         
            </pollingDuplexBinding>         
            <netTcpBinding>          
                <binding name="PubSub">           
                    <security mode="None"/>           
                </binding>           
             </netTcpBinding>           
        </bindings>         
        <services>         
            <service behaviorConfiguration="Microsoft.Samples.Silverlight.PollingDuplex.Service.PubSubServiceBehavior"         
                               name="Microsoft.Samples.Silverlight.PollingDuplex.Service.PubSubService">         
                <endpoint address="" binding="pollingDuplexBinding" bindingConfiguration="PubSub"         
                                       contract="Microsoft.Samples.Silverlight.PollingDuplex.Service.IPubSub"/>         
<endpoint address="" binding="netTcpBinding" bindingConfiguration="PubSub"          
                                       contract="Microsoft.Samples.Silverlight.PollingDuplex.Service.IPubSub"/>         
                <endpoint address="mex" binding="mexHttpBinding" contract="IMetadataExchange"/>         
            </service>         
        </services>         
    </system.serviceModel> 

```
  

This change will add a new net.tcp endpoint to the existing WCF pub\sub service. The net.tcp endpoint will exist side by side with an endpoint based on the HTTP polling duplex protocol, enabling clients to communicate with the service over either of the protocols. Metadata exchange endpoint will enable automatic generation of a client proxy for either of the protocols.   

### Publishing in IIS7  

The development web server built into Visual Studio 10 does not support the net.tcp protocol, so you will need to deploy the web application to IIS7 to run the sample. You can use the Publish feature available for web application projects in Visual Studio to deploy the new web application to IIS7. In this article I am assuming the service will be deployed to http://localhost/pubsub.  

Next, you must make sure the web application is properly configured for accepting net.tcp connections.  

### Enable net.tcp activation in IIS7  

One important feature of IIS is the ability to activate a web application when an HTTP request targeting this application is received. Similar feature exists in IIS7 for net.tcp requests, but it may not be enabled by default depending on your system configuration. To make sure the feature is enabled or to turn it on, go to Control Panel \| Programs \| Turn windows features on or off and make sure WCF non-http activation is enabled:  

 ![Enable non-http activation in Windows 7](http://lh3.ggpht.com/_NUp_nWDyyvI/SwRbED8aWxI/AAAAAAAABDE/KByUmE-8MiM/EnableNonHttpActivation_thumb2.png?imgmax=800)   

If you are running Windows 2008 server, the same feature is available through Server Manager.   

To find out if net.tcp activation is enabled on your machine, go to the command line and run the following command:  

```

sc query NetTcpActivator

```
    

The output should indicate the local service is running.  

### Enable net.tcp protocol in IIS7  

Web applications in IIS7 can be selective about the protocols they support. When you create a new web application using the Publish feature of Visual Studio 10, it is configured to support HTTP protocol only and must be explicitly configured to enable net.tcp traffic as well. This can be done through Internet Information Services Manager. Go to the web application you have created (localhost/pubsub), choose Manage Application \| Advanced Settings, and make sure net.tcp protocol is listed in Enabled protocols:  

 ![Enable net.tcp protocol for a web application](http://lh5.ggpht.com/_NUp_nWDyyvI/SwRbErvSc6I/AAAAAAAABDM/PtZr1v5jpZo/image_thumb4.png?imgmax=800)   

Furthermore, the web site within which the web application resides must provide net.tcp protocol binding that allows connections using desired TCP port numbers. The sample uses TCP port 4502, so the binding specification must at minimum allow this port. Go to the web site containing your web application (usually the DefaultWebSite), choose Edit Bindings, and make sure the net.tcp binding allows port 4502:  

 ![II7 binding configuration for net.tcp](http://lh4.ggpht.com/_NUp_nWDyyvI/SwRbGPLTjzI/AAAAAAAABDU/BSRhkiJ426Q/image_thumb7.png?imgmax=800)    

To verify the WCF service is set up to accept net.tcp connections, try navigating the to the service URL in the web browser (e.g. http://localhost/pubsub/PubSubService.svc). There should be no errors and the help page for the WCF service should be displayed.   

### Allow Silverlight applications to communicate over TCP  

For a Silverlight application to create a TCP connection to a backend server, the server must explicitly allow such a connection (this is a measure to prevent cross-domain security exploits). This is done by exposing an XML policy file over HTTP at the root of the domain where the Silverlight application is hosted, as documented at [Network Security Access Restrictions in Silverlight](http://msdn.microsoft.com/en-us/library/cc645032(VS.95).aspx). This is the same policy file that is also used to allow cross-domain HTTP requests from Silverlight applications, but its content is augmented to add policy for TCP connections as well.   

For example, to allow all Silverlight applications to open TCP connections on ports 4502-4530 to the machine, create a clientaccesspolicy.xml file with the following content and host at the root of the document directory of your IIS server (typically c:\inetpub\wwwroot):  

```
<?xml version="1.0" encoding="utf-8"?>        
<access-policy>         
  <cross-domain-access>         
    <policy>         
      <allow-from http-request-headers="*">         
        <domain uri="*" />         
      </allow-from>         
      <grant-to>         
        <resource path="/" include-subpaths="true" />         
        <socket-resource port="4502-4530" protocol="tcp" />         
      </grant-to>         
    </policy>         
  </cross-domain-access>         
</access-policy> 

```
  

Open you web browser and navigate to http://localhost/clientaccesspolicy.xml to verify it will be accessible to Silverlight applications. The permit-all policy above allows unrestricted cross domain calls for both HTTP and TCP protocol over ports 4502-4530.   

The section below is applicable to Silverlight 4 Beta. As of Silverlight 4 RC (and Silverlight 4 RTM), the socket policy for WCF clients using the net.tcp protocol is obtained over HTTP as opposed to TCP port 943 – please see the preceding paragraph.            
            
          

For a Silverlight application to create a TCP connection to a backend server, the server must explicitly allow such a connection (this is a measure to prevent cross-domain security exploits). This is done by exposing a TCP socket policy over TCP port 943 as documented at [Network Security Access Restrictions in Silverlight](http://msdn.microsoft.com/en-us/library/cc645032(VS.95).aspx).           

There is an online project template for Visual Studio 10 which makes this task simple by creating a windows console application serving a TCP socket policy file. The easiest way to add this project to a Visual Studio 10 solution is by searching for “silverlight tcp” in the online templates through the Add New Project dialog in Visual Studio 10:          

 ![Silverlight TCP Socket Policy online project template in Visual Studio 10](http://lh3.ggpht.com/_NUp_nWDyyvI/SwRbHAKiW5I/AAAAAAAABDc/aEJmKrnbF34/image_thumb16.png?imgmax=800)          

The default socket policy included in the project template allows all Silverlight applications to connect to ports 4502-4534, which is the entire range of ports available to Silverlight applications. The policy can be customized if necessary. You can also [access the Silverlight TCP Socket Policy project template outside of Visual Studio](http://visualstudiogallery.msdn.microsoft.com/en-us/c4534af5-e864-42c2-b351-094593864e78).            

Remember to start the socket policy server application on the machine where the WCF service is hosted before running the Silverlight client! 

### Creating a service proxy to the WCF net.tcp pub/sub service  

Creating a service proxy to a WCF net.tcp service is easy with the Add Service Reference feature of Visual Studio 10. The process is essentially the same as adding a service proxy to an HTTP based request/response or duplex service.   

One feature the WCF net.tcp proxy offers beyond what HTTP polling duplex supports is integration with configuration file. During proxy generation, endpoint and binding information will be stored in the ServiceReferences.ClientConfig file, which enables the service address and other binding details to be controlled declaratively through config:   

```
<configuration>        
    <system.serviceModel>         
        <bindings>         
            <customBinding>         
                <binding name="NetTcpBinding_IPubSub">         
                    <binaryMessageEncoding />         
                    <tcpTransport maxReceivedMessageSize="2147483647" maxBufferSize="2147483647" />         
                </binding>         
            </customBinding>         
        </bindings>         
        <client>         
            <endpoint address="net.tcp://localhost:4502/pubsub/PubSubService.svc"         
                binding="customBinding" bindingConfiguration="NetTcpBinding_IPubSub"         
                contract="Proxy.IPubSub" name="NetTcpBinding_IPubSub" />         
        </client>         
    </system.serviceModel>         
</configuration> 

```
  

An instance of the proxy using net.tcp protocol can be created in code by referring to the named endpoint in the configuration file:  

```
PubSubClient client = new PubSubClient("NetTcpBinding_IPubSub"); 

```
  

The same proxy class can be used with HTTP polling duplex protocol, but the binding and address must be provided in code (we are working on providing configuration support for HTTP polling duplex before Silverlight 4 release):   

```
PubSubClient client = new PubSubClient(        
    new PollingDuplexHttpBinding(),         
    new EndpointAddress(new Uri(App.Current.Host.Source + "/../../PubSubService.svc"))); 

```
  

### Running the pub/sub sample  

I have described key steps necessary to develop and deploy a Silverlight 4 application using the new WCF net.tcp protocol. [The full Visual Studio 10 Beta 2 solution of this Silverlight 4 sample is available for download](http://janczuk.org/code/samples/pubsubsamplenettcp.zip). The sample contains a pub\sub WCF server exposed over two endpoints: one using the HTTP polling duplex binding available in Silverlight 2 and 3, and the other using the WCF net.tcp protocol newly supported in Silverlight 4. There are three types of clients:  

* A pub\sub Silverlight 4 client that offers a choice of HTTP polling duplex or net.tcp protocols.  
* A pub\sub AJAX client that communicates with the server using the HTTP polling duplex protocol.  
* A publisher Silverlight 4 client that offers a choice of HTTP polling duplex or net.tcp protocols.  
  

So what does this sample demonstrate? You can have a Silverlight 4 publisher using net.tcp protocol to publish messages to a topic managed by a WCF pub\sub service, and three pub\sub clients consuming these notifications seamlessly using net.tcp or HTTP polling duplex in Silverlight, or HTTP polling duplex in AJAX. I have added a fragment of Dante’s Divine Comedy if you think publishing stock quotes is too boring.   

 ![Publisher client using WCF net.tcp in Silberlight 4 running in Chrome](http://lh6.ggpht.com/_NUp_nWDyyvI/SwRbH-ESydI/AAAAAAAABDo/osk9phvwyfc/image_thumb%5B4%5D.png?imgmax=800)  ![Pub\sub client using WCF net.tcp protocol in Silverlight 4 running in Internet Explorer](http://lh5.ggpht.com/_NUp_nWDyyvI/SwRbIwvZqHI/AAAAAAAABDw/izoCXqspjek/image_thumb%5B5%5D.png?imgmax=800)  

 ![Pub\sub client using WCF HTTP polling duplex protocol in Silverlight 4 running in Mozilla Firefox](http://lh6.ggpht.com/_NUp_nWDyyvI/SwRbJs2PrOI/AAAAAAAABD4/p0nb1YGD_LA/image_thumb%5B6%5D.png?imgmax=800) ![Pub\sub client using WCF HTTP polling duplex protocol in AJAX running in Internet Explorer](http://lh4.ggpht.com/_NUp_nWDyyvI/SwRbKct6mdI/AAAAAAAABEA/MmS7Y6M3j0E/image_thumb%5B7%5D.png?imgmax=800)  

You can read more about the HTTP polling duplex and AJAX aspects of the sample in several of my previous posts. For more in-depth look at net.tcp, check out the most recent post on [WCF net.tcp in Silverlight 4](http://tomasz.janczuk.org/2009/11/wcf-nettcp-protocol-in-silverlight-4.html).  }