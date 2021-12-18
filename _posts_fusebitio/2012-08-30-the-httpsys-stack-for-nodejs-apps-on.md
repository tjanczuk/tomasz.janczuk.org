---
post_title: The HTTP.SYS stack for node.js apps on Windows
date: 2012-08-30
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: the-http.sys-stack-for-node.js-apps-on-windows
post_date_in_url: true
post_og_image: site
post_excerpt: Tomek on Software - shaken, not stirred
tags:
  - post
---




I have been working lately on the [httpsys](https://github.com/tjanczuk/httpsys) project, which offers an HTTP.SYS based HTTP[S] stack for node.js applications running on Windows.   

The basic idea is to replace the HTTP stack built into node.js with HTTP.SYS while preserving the shape and behavior of HTTP APIs in node.js to the extent possible. The expected outcome is that self-hosted node.js applications on Windows can enjoy better performance and other benefits offered by native, kernel mode HTTP.SYS implementation that the built-in HTTP stack does not support (e.g. kernel mode output caching, port sharing). The work had been done as a native node.js module that integrates tightly the HTTP.SYS functionality with the IO mechanisms of libuv, using the same IO completion port that underlies all async operations of node.js on Windows. No changes in the node or libuv were necessary.   

The httpsys is applicable to node.js applications self-hosted on Windows and does not affect the way hosting of node.js application works in IIS. The latter hosting mechanism is supported by [iisnode](http://tomasz.janczuk.org/2011/08/hosting-nodejs-applications-in-iis-on.html).  

In this post I am taking an early look at the performance of the httpsys module compared to the built-in HTTP stack in node. Note that this is a very early cut of the httpsys module that did not see much testing or performance optimization.   

### Who killed the butler (aka summary)  

Here is the gist of the observations. Before you roll out the cannons, please read the rest of this post however.   

* up to 2x improvement in HTTP throughput  
* up to 5.5x improvement in HTTPS throughput  
* up to 13x improvement in HTTP throughput with use of HTTP.SYS output caching [1]  
* up to 25% reduction of HTTP latency  
* up to 85% reduction of HTTPS latency  
  

[1] The measurement was network-bound at 50% server CPU, the potential benefit is therefore likely much larger.   

### The test environment  

All measurements were conducted in a lab with a private network (i.e. no external networking interference); this is the setup:  

* server: Intel Xeon L5520 @ 2.27GHz (4 cores), Windows 2008 R2 x64  
* client: Intel Xeon L5520 @ 2.27GHz (16 cores), Windows 2008 R2 x64  
* 1GB network  
* node.js 0.8.7 x64 with hotfixes for cluster functionality in the built-in HTTP stack thanks to @bertbelder (SHA bea0ac)  
* httpsys 0.2.0 x64  
* WCAT configured with 500 virtual clients, 10 second warmup, and 30 second test duration  
* [a 'Hello, world' node.js application](https://github.com/tjanczuk/httpsys/tree/master/test/performance)  
* a self-signed 1024 bit X.509 certificate  
  

### Scenarios measured  

The measured variants were a cross product of the following parameters:  

(built-in, HTTP.SYS) x (1 process, cluster) x (HTTP/keep-alive, HTTP/close, HTTPS/keep-alive, HTTPS/reconnect, HTTPS/full)  

as well as:  

(HTTP.SYS with 1 second output caching) x (1 process) x (HTTP/keep-alive, HTTP/close, HTTPS/keep-alive, HTTPS/reconnect, HTTPS/full)  

The measurement of clustered deployment with HTTP.SYS output caching was not conducted because a single node.exe process was sufficient to saturate the CPU on the server in that case (HTTP.SYS is multithreaded).   

### The outliers  

There are a few outliers in the data collected that need further investigation and will likely affect the results once fixes or adjustments are made:  

1. The throughput of a one process deployment with the built-in HTTP stack and Connection: close is much lower than expected. Initial investigation shows a large number of failed TCP connection attempts from the client. It is likely that once the issue is resolved, the throughput will substantially increase.  
2. The throughput of the HTTP.SYS stack in case of Connection: close with full SSL handshake is lower than expected (2x lower than corresponding throughput of OpenSSL in node.js). The issue remains to be investigated.  
3. The variation of HTTP/keep-alive using httpsys with 1 second output caching was network bound; the server only ever reached 50% utilization. With more bandwidth at hand the throughput would likely substantially increase.  
  

### Four reasons you should distrust this post (aka disclaimer)  

1. The measurements were taken using a node.js v0.8.7 and httpsys v0.2.0, and both of these technologies continue to rapidly move forward. It is likely that the same measurements done on the most current versions would yield different results.  
2. The application measured (‘Hello, world’) hardly represents a real world workload.  
3. I have developed [httpsys](https://github.com/tjanczuk/httpsys), why would you trust me to objectively talk about its performance?  
4. Buts. The area of performance measurements yields itself very well to “but”-type of arguments. But the scenario is not real. But the machine is not like a machine I like better. But the payload is too small/large. You get the idea. So in the spirit of transparency [you can see the test code](https://github.com/tjanczuk/httpsys/tree/master/test/performance), you can repeat the measurements, you can add your own tests (and make a pull request) if you think you have a useful scenario to measure.  
  

### Show me the data already    

And now on to the grand finale.   

Throughput results in a non-clustered deployment show the httpsys stack performing better across all scenarios measured, with the 1 second output caching variation showing rather dramatic throughput results at mere 50% CPU utilization (the measurement was network bound). Note that the built-in stack in the HTTP/close variation was subject to an issue whereby a relatively large rate of failed TCP connection attempts were experienced. Once this issue is investigated and fixed the result will likely substantially improve (this is the 3752 number below).   

 ![image](http://chart.apis.google.com/chart?chxl=1:%7CHTTP%2Fkeep-alive%7CHTTP%2Fclose%7CHTTPS%2Fkeep-alive%7CHTTPS%2Freconnect%7CHTTPS%2Ffull%7C2:%7C%5Breq%2Fs%5D&chxp=2,50&chxr=0,0,170000&chxs=0,676767,9.833,0,lt,676767%7C1,676767,11.5,0,l,00000000&chxt=y,x,y&chbh=a,4,25&chs=600x500&cht=bvg&chco=80C65A,FF9900,AA0033&chds=0,170000,0,170000,0,170000&chd=t:12461,3752,5651,1357,532%7C17364,15686,14495,7382,747%7C159902,35624,109148,9824,742&chdl=built-in%7CHTTP.SYS%7CHTTP.SYS+%2B+caching&chdlp=b&chma=0,0,0,2%7C0,7&chtt=Throughout+(larger+is+better)%2C+1+node.exe+process&chts=000000,11.5&chm=N,676767,0,-1,11%7CN,676767,1,-1,11%7CN,676767,2,-1,11)  

Latency results show the 95-percentile latency of receiving the last byte of the response. The httpsys module shows lower latency across all scenarios measured. In case of the HTTP/close scenario with the built-in module the latency was not measured at all due to aforementioned issue.   

 ![image](http://chart.apis.google.com/chart?chxl=1:%7CHTTP%2Fkeep-alive%7CHTTP%2Fclose%7CHTTPS%2Fkeep-alive%7CHTTPS%2Freconnect%7CHTTPS%2Ffull%7C2:%7C%5Bms%5D&chxp=2,50&chxr=0,0,1500&chxs=0,676767,9.833,0,lt,676767%7C1,676767,11.5,0,l,00000000&chxt=y,x,y&chbh=a,4,25&chs=600x500&cht=bvg&chco=80C65A,FF9900,AA0033&chds=0,1500,0,1500,0,1500&chd=t:17,-1,36,672,1440%7C13,25,15,101,1168%7C3,18,5,136,1040&chdl=built-in%7CHTTP.SYS%7CHTTP.SYS+%2B+caching&chdlp=b&chma=0,0,0,4%7C0,7&chtt=Latency+(smaller+is+better)%2C+1+node.exe+process&chts=000000,11.5&chm=N,676767,0,-1,11%7CN,676767,1,-1,11%7CN,676767,2,-1,11)  

The throughput results in a cluster deployment that saturates the server CPU shows a few interesting data points. First, the throughput of HTTP/keep-alive variation is roughly similar between the built-in and HTTP.SYS stack. One possible explanation of this is that most of the advantage the HTTP.SYS solution shows in non-clustered variants stems from more efficient TCP connection handling; once a TCP connection is established, most other aspects affecting performance in this case are equal between these two stacks.   

Another interesting observation is the large benefit of HTTP.SYS over the built-in stack is HTTPS/keep-alive variant. It seem to indicate that steady-state symmetric cryptography over established SSL connection is much more efficiently implemented in HTTP.SYS than in OpenSSL.   

The HTTPS with Connection: close and full SSL handshake is the only variant across all measured in which HTTP.SYS performs worse than the built-in HTTPS stack in node.js. This was called out previously as an outlier than needs to be investigated.   

 ![image](http://chart.apis.google.com/chart?chxl=1:%7CHTTP%2Fkeep-alive%7CHTTP%2Fclose%7CHTTPS%2Fkeep-alive%7CHTTPS%2Freconnect%7CHTTPS%2Ffull%7C2:%7C%5Breq%2Fs%5D&chxp=2,50&chxr=0,0,36000&chxs=0,676767,9.833,0,lt,676767%7C1,676767,11.5,0,l,00000000&chxt=y,x,y&chbh=21,4,54&chs=600x500&cht=bvg&chco=80C65A,FF9900&chds=0,36000,0,36000&chd=t:34003,10225,14941,1507,1338%7C34760,20324,27834,8178,754&chdl=built-in%7CHTTP.SYS&chdlp=b&chma=0,0,0,1%7C0,13&chtt=Throughput+(larger+is+better)%2C+cluster+of+4+node.exe+processes&chts=000000,11.5&chm=N,676767,0,-1,11%7CN,676767,1,-1,11%7CN,676767,2,-1,11)      

### Conclusions  

It appears that substituting the built-in HTTP stack in node.js with one based on HTTP.SYS using the httpsys module can deliver substantial performance improvements. In addition to performance, HTTP.SYS enables port sharing (ability for two node.exe processes to listen on a subset of the URL space on a single TCP port), and additional performance boost of using HTTP.SYS kernel mode output cache.   

So install [httpsys](https://github.com/tjanczuk/httpsys) today, play with it, and let me know what you think.   }