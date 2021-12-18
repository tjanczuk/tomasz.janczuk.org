---
post_title: CPU bound workers for node.js applications using in-process .NET and OWIN
date: 2013-02-24
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: cpu-bound-workers-for-node.js-applications-using-in-process-.net-and-owin
post_date_in_url: true
post_og_image: site
post_excerpt: Tomek on Software - shaken, not stirred
tags:
  - post
---




*Note: this post is related to the 0.4.0 version of the owin project. The project has since been renamed to edge.js and has seen major improvements. [Visit edge.js](http://tomasz.janczuk.org/2013/03/run-c-and-nodejs-code-in-process-with.html) for the latest.*  

The [owin](https://github.com/tjanczuk/owin) project allows hosting .NET 4.5 code in node.js applications running on Windows. In my other posts I described how owin can be used to [implement express.js request handlers and connect middleware in .NET](http://tomasz.janczuk.org/2013/02/hosting-net-code-in-nodejs-applications.html) as well as help in [accessing SQL from node.js applications using in-process CLR/ADO.NET](http://tomasz.janczuk.org/2013/02/access-ms-sql-from-nodejs-application.html). In this post I will focus on running CPU-bound computations within the node.js application using owin.   

### The problem  

Node.js is not well suited to executing blocking, CPU bound workloads. The distinguishing design trait of node.js is its single-threaded, event-loop based architecture. It allows you to very efficiently handle IO-bound workloads. But executing CPU-bound computations blocks the event loop and makes the application unresponsive to subsequent IO events. For example, you can very efficiently accept a streaming upload of an image over HTTP, which is an IO bound workload. But you cannot subsequently run face recognition algorithm on this image, as this is a CPU-bound operation. It would prevent the node.js application from processing other image uploads.    

Node.js applications typically process CPU-bound workloads by delegating the processing to an external process or service. This involves crossing the process boundary and incurs additional latency.   

### In-process workloads in node.js using owin  

The [owin](http://tomasz.janczuk.org/2013/02/hosting-net-code-in-nodejs-applications.html) module allows running CPU-bound computations implemented in .NET in-process with the node.js application without blocking the node.js event loop. The CPU bound workloads execute on CLR threads separate from the singleton V8 thread in the node executable. Owin facilitates data marshaling between node.js and .NET components of the application, as well as reconciles the threading models of the two.   

Get started by importing the owin module:  

{% highlight text linenos %}
   npm install owin@0.4.0

{% endhighlight %}





Then implement your CPU-bound workload in .NET as follows and save the result in Startup.cs file:

{% highlight csharp linenos %}
namespace CalculateBudget  
{  
    public class Startup : Owinjs.Worker  
    {  
        protected override IDictionary<string, string> Execute(IDictionary<string, string> input)  
        {  
            int revenue = int.Parse(input["revenue"]);  
            int cost = int.Parse(input["cost"]);  
            int income = revenue - cost;  
  
            Thread.Sleep(5000); // pretend it takes a long time  
  
            return new Dictionary<string, string> { { "income", income.ToString() } };  
        }  
    }  
}  

  

{% endhighlight %}



Notice the signature of the Execute method, which is the contract for specifying input to and returning results from your CPU-bound workload. This mechanism allows you to pass in a dictionary of strings and return another one. These dictionaries are marshaled from and into JavaScript objects in the node.js application. 

The Execute method is called on a CLR thread allocated by the Owinjs.Worker base class. The base class implements a thin adapter layer between the [OWIN](http://owin.org/) interface the owin module is built around and the signature of the Execute method above. 

Now compile the Startup.cs into CalculateBudget.dll and reference the Owinjs.dll that comes with the owin module:

{% highlight text linenos %}
copy node_modules\owin\lib\clr\Owinjs.dll  
csc /target:library /r:Owinjs.dll /out:CalculateBudget.dll Startup.cs
  

{% endhighlight %}



Lastly, implement a node.js application which invokes the CPU-bound computation using the owin module, and save it to test.js file:

{% highlight javascript linenos %}
var owin = require('owin')  
  
console.log('Starting long running operation...');  
owin.worker(  
    'CalculateBudget.dll',  
    { revenue: 100, cost: 80 },  
    function (error, result) {  
        if (error) throw error;  
        console.log('Result of long running operation: ', result);  
    }  
);  
  
setInterval(function () {   
    console.log('Node.js event loop is alive!')  
}, 1000);
  

{% endhighlight %}



This application starts the CPU-bound computation using the OWIN module, and then initiates an interval that will print out a message on the screen every second. When you start this application with

{% highlight text linenos %}
node test.js
  

{% endhighlight %}



You will see the following output:

{% highlight text linenos %}
C:\projects\owin_test>node test.js  
Starting long running operation...  
Node.js event loop is alive!  
Node.js event loop is alive!  
Node.js event loop is alive!  
Node.js event loop is alive!  
Node.js event loop is alive!  
Result of long running operation:  { income: '20' }  
Node.js event loop is alive!  
Node.js event loop is alive!
  

{% endhighlight %}


Notice how the interval is able to print out messages between the time the CPU-bound worker is started and it returns the result. This proves that the node.js event loop remains responsive while the CPU-bound computation takes place on a separate CLR thread. 



### More

Visit the project page at [https://github.com/tjanczuk/owin](https://github.com/tjanczuk/owin) for the latest bits. [Feedback](https://github.com/tjanczuk/owin/issues) and pull requests welcome.

Check out related posts:

[Implement express.js request handlers and connect middleware using in-process hosted CLR/.NET and OWIN](http://tomasz.janczuk.org/2013/02/hosting-net-code-in-nodejs-applications.html) 

    
[SQL access from node.js applications using in-process CLR/ADO.NET and OWIN](http://tomasz.janczuk.org/2013/02/access-ms-sql-from-nodejs-application.html)  }