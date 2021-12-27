---
tags: ['post']
post_og_image: 'site'
date: '2013-02-23'  
post_title: Hosting .NET code in node.js applications using OWIN
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: hosting-.net-code-in-node.js-applications-using-owin
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




*Note: this post is related to the 0.4.0 version of the owin project. The project has since been renamed to edge.js and has seen major improvements. [Visit edge.js](http://tomasz.janczuk.org/2013/03/run-c-and-nodejs-code-in-process-with.html) for the latest.*  

The [owin](https://github.com/tjanczuk/owin) project allows hosting .NET 4.5 code in node.js applications running on Windows. The goal of the project is to enable or simplify application scenarios which are hard or impossible to achieve with node.js alone, in particular:  

* **implementing express.js handlers and connect middleware** **in .NET 4.5** to leverage existing .NET components, frameworks, and tools in node.js web applications without crossing the process boundary,  
* **implementing CPU-bound workloads (workers) in-process** in node.js applications by executing blocking .NET code on the CLR thread pool and marshalling the results back to the node.js event loop upon completion,  
* **simplifying access to Windows specific functionality in node.js applications** by enabling the use of CLR languages (C#) and .NET Framework instead of implementing native node.js modules in C/C++ and Win32.  
  

Owin is a native node.js module implemented in C++\CLI. It hosts OWIN handlers ([http://owin.org/](http://owin.org/)) written in .NET 4.5 and exposes them to a node.js application. It also provides a connect wrapper around OWIN interface. This allows .NET OWIN handlers to be plugged in directly into the express pipeline either as connect middleware or as request handlers. The owin module takes care of marshaling data between V8 and CLR heapsand reconciling threading models of the two.  

In this post I will show a simple *Hello, world* experience of using the [owin](https://github.com/tjanczuk/owin) module. Look for subsequent posts that will deal with more specific topics, in particular:  

[CPU bound workers for node.js applications using in-process .NET and OWIN](http://tomasz.janczuk.org/2013/02/cpu-bound-workers-for-nodejs.html)       
[SQL access from node.js applications using in-process CLR/ADO.NET and OWIN](http://tomasz.janczuk.org/2013/02/access-ms-sql-from-nodejs-application.html)  

### Hello, world  

You need Windows x64 with node.js 0.8.x x64 installed (the module had been developed against node.js [0.8.19](http://nodejs.org/dist/v0.8.19/)). You also need [.NET Framework 4.5](http://www.microsoft.com/en-us/download/details.aspx?id=30653) on the machine.   

Start by importing the express.js and owin modules from NPM:  

```

   npm install express  
npm install owin@0.4.0

```


Then implement your [OWIN](http://owin.org/) handler in C# and save it to Startup.cs file:

```
using System;  
using System.Collections.Generic;  
using System.IO;  
using System.Text;  
using System.Threading.Tasks;  
  
namespace OwinHelloWorld  
{  
    public class Startup  
    {  
        public Task Invoke(IDictionary<string, object> env)  
        {  
            env["owin.ResponseStatusCode"] = 200;  
            ((IDictionary<string, string[]>)env["owin.ResponseHeaders"]).Add(  
                "Content-Type", new string[] { "text/html" });  
            StreamWriter w = new StreamWriter((Stream)env["owin.ResponseBody"]);  
            w.Write("Hello, from C#. Time on server is " + DateTime.Now.ToString());  
            w.Flush();  
  
            return Task.FromResult<object>(null);  
        }  
    }  
}

```


Compile the Startup.cs file to OwinHelloWorld.dll (by convention the assembly name should match the namespace of the Startup class):

```

csc /target:library /out:OwinHelloWorld.dll Startup.cs
  

```


Finally implement your express.js application and save it to server.js file. The application imports the owin module and uses it create and register an express.js request handler created around the the OwinHelloWorld.dll you just compiled: 

```
var owin = require('owin')  
    , express = require('express');  
  
var app = express();  
app.use(express.bodyParser());  
app.all('/jazz', owin('OwinHelloWorld.dll'))  
app.all('/rocknroll', function (req, res) {  
    res.send(200, 'Hello from JavaScript! Time on server ' + new Date());  
});  
  
app.listen(3000);
  

```


Note that the OwinHelloWorld.dll must be in the current directory, or a full path to the DLL must be specified in the call to the owin() function. 

Now run the node.js server:

```

node server.js
  

```


Finally, navigate to the http://localhost:3000/rocknroll URL. As expected, you get back the response from the JavaScript handler:

 ![rocknroll](http://lh5.ggpht.com/-YOMzn39PGjY/USmSt2VcqdI/AAAAAAAADZs/HTNBUQLwU7E/rocknroll_thumb%25255B1%25255D.png?imgmax=800) 

Then navigate to http://localhost:3000/jazz. You will get a response from your .NET handler:

 ![jazz](http://lh4.ggpht.com/-3Oe31uLi0cQ/USmSusqcsrI/AAAAAAAADZ8/XNY_UkpFRZE/jazz_thumb%25255B1%25255D.png?imgmax=800) 

### Debugging

You can debug the .NET code running within the node.exe process using Visual Studio or other debuggers for managed code. Since the node.exe process contains both native and managed code, when attaching the debugger to the process you must indicate you want to debug managed code:

 ![debug](http://lh5.ggpht.com/-JkoYc4xyyKk/USmSwWGeJMI/AAAAAAAADaM/XOVlECi6sbs/debug_thumb%25255B3%25255D.png?imgmax=800) 

From there, you can set breakpoints and step through your managed code as if it were a regular .NET application:

 ![debug1](http://lh6.ggpht.com/-lux6mnkXehg/USmSxjAiD4I/AAAAAAAADac/nvSDP1hRqpg/debug1_thumb%25255B2%25255D.png?imgmax=800) 

### Exceptions

Exceptions thrown by the .NET code are marshaled back to JavaScript application. Express.js will intercept these exceptions for you and return the exception text in the HTTP response, providing yet another mechanism to diagnose issues in the .NET code hosted in node.js:

 ![exception](http://lh4.ggpht.com/-GraSQ6f7poQ/USmSy8Pzr9I/AAAAAAAADas/VcPjn2sz-eA/exception_thumb%25255B2%25255D.png?imgmax=800)

### Memory footprint of hosting CLR in node.js

The memory footprint of node.exe will increase once the owin module loads CLR into the address space of the process. Memory consumption of the node.exe will vary depending on many factors specific to your application. To give you a general idea of the impact, here are some working set numbers:
<table><tr><td>The server.js application above without the owin module imported and without the .NET handler registered (i.e. pure JavaScript express.js application)</td><td>

19 220 KB</td></tr><tr><td>The server.js application above (i.e. one OWIN handler)</td><td>27 908 KB</td></tr><tr><td>The server.js application above with two OWIN handlers</td><td>27 940 KB</td></tr></table>


In the situation above one pays a one-time cost of around 8.5 MB when including the owin module and the first OWIN handler. Importing the owin module is what causes the CLR to be loaded into the address space of the node.exe process.

Adding more .NET handlers incurs only a small incremental cost (e.g. 32KB above). This is because the CLR is already loaded into memory and the extra cost is only related to loading additional, handler specific managed assemblies.

### More

Visit the project page at [https://github.com/tjanczuk/owin](https://github.com/tjanczuk/owin) for the latest bits. [Feedback](https://github.com/tjanczuk/owin/issues) and pull requests welcome.

Check out related posts:

[CPU bound workers for node.js applications using in-process .NET and OWIN](http://tomasz.janczuk.org/2013/02/cpu-bound-workers-for-nodejs.html) 

    
[SQL access from node.js applications using in-process CLR/ADO.NET and OWIN](http://tomasz.janczuk.org/2013/02/access-ms-sql-from-nodejs-application.html)  }