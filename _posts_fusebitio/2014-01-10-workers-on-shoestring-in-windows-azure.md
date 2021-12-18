---
post_title: Workers on a shoestring in Windows Azure Web Sites
date: 2014-01-10
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: workers-on-a-shoestring-in-windows-azure-web-sites
post_date_in_url: true
post_og_image: site
post_excerpt: Tomek on Software - shaken, not stirred
tags:
  - post
---




### Hosting web apps in Azure, by the book  

Many web apps consist of web, worker, and storage components. The web component handles HTTP traffic from clients which results in new work items (e.g. uploaded pictures that need to be resized). The worker component performs the actual work independently from interactions between the client and web component. Web and worker components exchange state using some form of external storage (e.g. a database or a queue).   

 ![image](http://lh3.ggpht.com/-3O92YzpgQms/UtCbkDmWBsI/AAAAAAAAD4Q/u2wkmOXfbxQ/image_thumb%25255B2%25255D.png?imgmax=800)   

In the general case, the three components are deployed to separate server farms to accommodate different scalability, reliability, computing resource, and process lifetime requirements.   

For web applications like this hosted in Windows Azure, there is a natural mapping of the web, worker, and storage components onto Windows Azure concepts. The web component would be running in Windows Azure Web Sites, which is by far the most convenient way of hosting web tier code in Azure. The worker component would run as Hosted Service or a Virtual Machine. The storage component is not something you want to run yourself these days, unless you have a very compelling reason not to use one of the many hosted storage solutions available in Azure (MongoHQ, MongoLab, Azure Blob, Azure Table, SQL Azure, etc.). The details of storage farm management are abstracted away from you, and your app perceives storage as an endpoint to talk to sticking from a black box.   

Given all that, a web application hosted in Azure would look like this:   

 ![image](http://lh4.ggpht.com/-PMw5AB8dkHs/UtCblKe_adI/AAAAAAAAD4c/bjs0ILSPf6k/image_thumb%25255B5%25255D.png?imgmax=800)  

### Problem in paradise  

While using Azure to develop a web application like the one above, the experience gap between working on the web tier hosted in Windows Azure Web Sites and the worker tier running in Hosted Services becomes apparent and annoying very quickly.   

Web Sites support code deployment in seconds using git. Hosted services take minutes to update code and require it to be done from VS or Windows only command line tools. Web sites provide very convenient streaming logging feature. Getting logs out of a Hosted Service is brittle.   

As a developer, I would love to have a worker tier development experience match that offered by Windows Azure Web Sites. I want quick, git-based deployment for both web and worker code. I want to deploy from Mac or Windows without discrimination. I want my streaming logs available for both web and worker, or perhaps even unified.     

### Let’s break some rules  

To achieve my ideal development experience, I am going to run both web and worker tier code in Windows Azure Web Sites:  

 ![image](http://lh3.ggpht.com/-IYzb1VmcP4I/UtCbmLeymRI/AAAAAAAAD4s/AYlK3ImtVI0/image_thumb%25255B8%25255D.png?imgmax=800)   

In general this is a big no-no most of the time, but there is a class of web applications for which having a single deployment container for both web and worker code is not entirely unreasonable. Below are some guidelines to decide if this is a good fit for your app.  

The **resource consumption profile** of both web and worker tier should be sufficiently similar. Web tier workloads are typically IO bound: they accept HTTP requests, do some minimal processing, turn around and exchange some data with the storage tier, then respond to the client. Worker tier profiles vary from CPU bound, memory bound, to IO bound. It is reasonably safe to combine a web tier with a worker tier that is also IO bound. For example, your worker tier may be implementing a long running IO orchestration, coordinating processes across several distributed systems. If the resource consumption profiles of web and worker tiers were different, chances are high one or more classes of resources would go underutilized when the system is scaled out to handle the traffic.   

The worker tier must be implemented in a way that is compatible with the **process management** of your web tier. In Azure Web Sites, processes are running under IIS. They are only activated when HTTP requests arrive, and the recycling policy will terminate them in pre-configured circumstances, e.g. within 15 minutes of lack of HTTP activity. You must design your worker tier to be robust enough to withstand this recycling policy. You must also mitigate the lack of control over process activation (more on this in the next section).   

The benefits of running both web and worker in Windows Azure Web Sites are numerous and particularly relevant at active development phase:  

* Simplicity: the is only one artifact to deploy and manage.  
* Logging: streaming logging from both web and worker components is available in a unified form.  
* Deployment: git-deploy in seconds both web and worker code, and make the deployment atomic between web and worker tier.  
* Cross-platform: deploy from Mac or Windows  
* Configuration: quickly update configuration settings of web and worker using the same mechanism (app settings in Windows Azure Web Sites propagated as environment variables to web and worker processes).  
  

### Workers on a shoestring, the practice  

There is a number of considerations for hosting worker code in Windows Azure Web Sites that must be addressed.   

**Initializing your worker process and keeping it running**  

Processes running in Windows Azure Web Sites are managed by IIS. IIS itself can be [configured to start up a process on system startup and keep it always running](http://tomasz.janczuk.org/2013/07/application-initialization-of-nodejs.html). However, the configuration of IIS in Windows Azure Web Sites is different and locked: processes are only activated when an HTTP request arrives that targets a particular application. As a corollary, without an HTTP request the process will never run.   

Moreover, IIS in Windows Azure Web Sites is configured to terminate web processes for which no HTTP requests were received during a specific period (15 minutes by default, but the application has no control over this value). A new process will only be created when another HTTP request arrives.   

To have a worker process initialized and running most of the time in this environment one must:  

* Create the worker process as soon as the web process is initialized by IIS. While technically you can run the worker logic from within the web process, it is a good idea to have a process boundary between web and worker. This reduces cold startup latency of the initiating HTTP request, and also helps keep web and worker logic encapsulated in case you need to split worker from web tier later. Note that if you spawn a worker process from within a web process, they are still going to run in the same Windows job object and therefore be bound by the same process lifetime policy that IIS imposes. If IIS decides to terminate the web process given its recycling policy, the worker process will be terminated with it, no questions asked.  
* Send an HTTP request to the web application periodically to ensure the web process (and the worker process spawned by it) are running. You can use an external system to send these periodic HTTP requests, but since we are implementing workers on a shoestring, let’s hack another Windows Azure feature to do the job for us for free: Health Monitoring endpoints. Every Windows Azure Website can be configured with a Health Monitoring endpoint that Azure will periodically invoke to measure and report on latency of calls originating from various places in the world:        
        
 ![image](http://lh5.ggpht.com/-joaeZbtDFwc/UtCbnz6xtaI/AAAAAAAAD48/PX3_Zwx9prE/image_thumb%25255B19%25255D.png?imgmax=800)         
        
As it happens, Azure invokes these endpoints every 5 minutes:         
        
 ![image](http://lh6.ggpht.com/-Dl-Oych1pQc/UtCboiMnAkI/AAAAAAAAD5Q/VIna2YYHCsk/image_thumb%25255B20%25255D.png?imgmax=800)         
        
Given that you can define up to 2 monitoring endpoint per web application in Windows Azure Web Sites, and each of these endpoints can be called from up to 3 worldwide locations for monitoring purposes, the combined frequency of periodic HTTP calls to your web site should be sufficient to reduce the risk of your worker process being down at any point.  
  

  

  

  

**Dealing with recycling**  

If you run your worker code in Windows Azure Web Sites, you have no control over when your process is recycled. This should be no huge issue from the reliability standpoint, since your worker logic should be implemented to properly handle unexpected failures anyway (recycling is no different than any other unexpected event that causes your process to terminate).   

In practice, however, worker logic is often optimized for certain assumptions around typical process lifetime. For example, you may run for 30 minutes before committing in-memory results to durable storage if you assume failures are infrequent and you otherwise control the process lifetime.   

Given that you know your worker process is likely to be terminated by IIS more frequently, you should design around this assumption. Make your “transactions” smaller and commit often. This way when a worker process is created anew after being recycled, it can pick up from where it left off without loosing much work.   

**Dealing with unexpected worker termination**  

What should happen when your worker process unexpectedly terminates? Since it was spawned by the web tier process, that situation must be handled by the web tier code itself. You can either implement your own worker process lifetime policy within the web process code, or you can rely on the IIS policy for handling unexpected application process failures. Most of the time you probably don’t want to roll out your own process lifetime management mechanism where one already exists. Instead, when a web process detects termination of the worker process it spawned, the web process should just terminate itself and let IIS handle this situation. When a next HTTP request arrives, the web/worker process combo will be created anew.   

**Keep web and worker code separate**  

Once you grow out of the shoestring solution described here, you will need to separate your web and worker components into separate containers. To make this easy, it is best to minimize any interaction or shared state between the web and worker processes despite they run on the same machine. Having the durable storage be the only way for web and worker to exchange data makes it so much easier to separate them when the time comes.   

The only on-machine interaction between web and worker processes should be scoped to the web process spawning the worker process, and web process terminating itself upon unexpected worker process termination.   

**Limitations of scalability**  

The scalability mechanism of Windows Azure Web Sites really prevents reliable use of this shoestring mechanism on deployments involving more than 1 instance.   

When your worker logic needs to be scaled out to handle the workload, you must be able to say “I need 5 instances of workers now” and have all of the 5 instances running concurrently. This is not how Windows Azure Web Site scalability works. When you say “I need 5 web instances now”, Azure really interprets it as “up to 5 instances”. The actual number of instances that will be running depends on the incoming HTTP traffic. So unless your worker scalability needs are always proportional to the number of incoming HTTP requests, you are likely to run into a situation where worker processes cannot keep up with outstanding work.   

### Workers on a shoestring, Mobile Chapters case study  

I have successfully used the shoestring approach to run worker processes as part of the [Mobile Chapters](https://mobilechapters.com/) web application.   

At the core, the web application accepts a book manuscript upload, stores the file in a durable store, and let’s a worker process asynchronously convert the manuscript into mobile applications for iOS, Android, and Windows Phone. The overall conversion process can take between seconds and minutes, depending on the complexity and size of the manuscript. The process is mostly IO bound, coordinating data flow and state transitions between PhoneGap Build, Azure Blob Storage, and MongoDB.   

Another job the worker process performs is to periodically refresh data the mobile applications can later fetch by calling out to external services. This is scheduled to happen every 15 minutes or so, and according to logs from loggly it works as clockwork. So the mechanism described here also yields itself well to the implementation of lightweight web schedulers.   

 ![image](http://lh5.ggpht.com/-f_aocxENeZM/UtCbpg9O--I/AAAAAAAAD5c/fjJdH_uk-D0/image_thumb%25255B22%25255D.png?imgmax=800)   

The important part is the shoestring approach provides me as a developer with a superior experience compared to what I would have to endure if I hosted worker code in a Hosted Service, without compromising the functionality of the web application.   

Enjoy!  }