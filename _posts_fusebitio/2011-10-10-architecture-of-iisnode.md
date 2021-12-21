---
tags: ['post']
post_og_image: 'site'
date: '2011-10-10'  
post_title: Architecture of iisnode
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: architecture-of-iisnode
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




This post describes high level architecture and design of iisnode (as of version 0.1.8), a native Internet Information Services (IIS) module that enables hosting node.js applications in IIS on Windows. iisnode is an open source project available at [https://github.com/tjanczuk/iisnode](https://github.com/tjanczuk/iisnode).   

The diagram below shows key components of iisnode and the relationship between them. Please note this is current as of version 0.1.8 and subject to change going forward.   

 ![image](http://lh3.ggpht.com/-AqY1JT6EhDk/TpM-vzW8sbI/AAAAAAAAB10/s-y2jbOja3Y/image_thumb%25255B2%25255D.png?imgmax=800)  

**CNodeHttpModuleFactory** implements the IHttpModuleFactory extensibility point of IIS in order to participate in the [IIS processing pipeline](http://learn.iis.net/page.aspx/101/introduction-to-iis-architecture/#HTTP).  The module factory is registered for the RQ_EXECUTE_REQUEST_HANDLER stage of the pipeline to enable node.js applications to process HTTP requests and generate HTTP responses. CNodeHttpModuleFactory crates an instance of CNodeHttpModule for every HTTP request processed by the handler.   

**CNodeHttpModule** is a thin wrapper around the logic of handling HTTP requests targeting a node.js application. Most of the per-request state is maintained in an instance of **CNodeHttpStoredContext**, while most of the processing logic is implemented in static methods of **CProtocolBridge**.   

**CNodeHttpStoredContext** maintains a state machine governing the processing of a single HTTP request/response under iisnode control. State transitions are triggered by events handled by CProtocolBridge. **CProtocolBridge** is a set of static callbacks handling asynchronous events related to the lifetime of an HTTP request/response. CProtocolBridge bridges between the HTTP handling APIs exposed by IIS on one side and the HTTP over named pipes protocol used to communicate with the node.exe process on the other. Its implementation is fully asynchronous – it does not perform thread blocking operations.   

The key top level component in iisnode is **CNodeApplicationManager**. It is a singleton responsible for managing several node.js applications hosted within a single IIS web application (possibly alongside other application types, e.g. ASP.NET, WCF, PHP, or static content). It performs activation of CNodeApplication instances when the first HTTP request targeting a particular node.js application arrives (a node.js application refers to a single *.js file that is the main entry point to the app). It then maintains a set of CNodeApplication instances and dispatches subsequent HTTP requests among them based on the physical path of the *.js file that an HTTP request targets. CNodeApplicationManager also maintains CAsyncManager and CFileWatcher.   

**CAsyncManager** maintains an IO thread pool for processing asynchronous callbacks from all applications managed by CNodeApplicationManager. The number of threads is configurable. Asynchronous callbacks are associated with IO completion port completions and related to communication between iisnode and node.exe over named pipes. Some asynchronous completions are timer-based and associated with named pipe connection retries.   

**CFileWatcher** is responsible for detecting changes in the *.js files containing node.js applications in support of the auto-update feature. When a *.js file changes, an existing CNodeApplication instance running it is gracefully retired, and a new CNodeApplication instance is created in its place. Currently active HTTP requests are allowed to finish using the old version of the application (within a configurable grace period), while all new HTTP requests are dispatched to the new version of the application. File watcher uses the efficient, asynchronous file change notification mechanisms for files on a local file system, or a periodic polling for files hosted on a network share.   

**CNodeApplication** is responsible for managing a single node.js application. (Several independent node.js applications can be present within a single IIS web application). It maintains **CPendingRequestsQueue**, a FIFO queue of HTTP requests pending processing by that application. Pending request queue is a temporary storage of HTTP requests that is designed to handle intermittent spikes in traffic targeting an application while all node.exe processes handling that application are too busy to accept a new request. When the pending request queue reaches its preconfigured maximum size, iisnode will start responding to all new requests targeting this application with a 503 (server too busy). CNodeApplication also maintains **CNodeProcessManager** responsible for managing several node.exe processes handling a single node.js application (a feature meant to enable utilization of several cores on multi core machines). It implements logic of creating new node.exe processes. It maintains a set of CNodeProcess instances and routes new requests among them. Currently it employs a round robin routing logic. CNodeProcessManager can spawn up to a preconfigured number of CNodeProcess instances per application.   

**CNodeProcess** manages a single node.exe process. It knows how to create an instance of the process using the path to node.exe provided in the iisnode section of web.config. When creating a new instance of the node.exe process, stdout and stderr are redirected to text files on disk in a location relative to the *.js file itself, hence enabling access to log files over HTTP. (See the section about [logging](http://tomasz.janczuk.org/2011/08/hosting-nodejs-applications-in-iis-on.html) for more details). CNodeProcess periodically flushes these handles to ensure the log files are refreshed at a preconfigured frequency. CNodeProcess can also detect termination of the node.exe process and responds by cleaning up all related resources (including closing any active HTTP requests handled by that process). Lastly, CNodeProcess keeps track of the set of HTTP requests currently processed by the node.exe process. Currently a single CNodeProcess will throttle work of node.exe by limiting the number of active requests it can handle at any any given time. If that quota has been reached for all CNodeProcess instances handling a given node.js application, requests will be queued up in the CNodePendingRequestsQueue handled by the associated CNodeApplication. CNodeProcess will dequeue a requests from the pending reqeusts queue only after it finishes processing one of the active requests.   }