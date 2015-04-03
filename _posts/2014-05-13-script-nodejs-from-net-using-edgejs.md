---
layout: post
title: Script Node.js from .NET using Edge.js
date: '2014-05-13T10:34:00.001-07:00'
author: Tomasz Janczuk
tags:
- JavaScript
- edgejs
- edge.js
- node.js
- .net
- edge
modified_time: '2014-05-13T10:34:49.196-07:00'
thumbnail: http://lh4.ggpht.com/-lBmw2OCGggE/U3JXqyHqjGI/AAAAAAAAD9A/AYyIabNqgwI/s72-c/DoubleEdge_thumb1.png?imgmax=800
blogger_id: tag:blogger.com,1999:blog-2987032012497124857.post-6322586821356382667
blogger_orig_url: http://tomasz.janczuk.org/2014/05/script-nodejs-from-net-using-edgejs.html
---




The latest release of the Edge.js project adds support for scripting Node.js from a .NET application. This enables you to leverage the power of Node.js ecosystem with the thousands of NPM modules from within a CLR application.   

[Learn more](http://tjanczuk.github.io/edge)       
[Get the all-inclusive Edge.js NuGet package](https://www.nuget.org/packages/Edge.js)  

You can now script Node.js code (not just JavaScript) within a .NET or ASP.NET web application written in C# or any other CLR language:  

 ![DoubleEdge](http://lh4.ggpht.com/-lBmw2OCGggE/U3JXqyHqjGI/AAAAAAAAD9A/AYyIabNqgwI/DoubleEdge_thumb1.png?imgmax=800)   

The Edge.js project existed for a while, but until now it only allowed scripting CLR code *from* a Node.js process on Windows, MacOS, and Linux. With the latest release, you can also script Node.js code from a CLR process.   

You can call .NET functions from Node.js and Node.js functions from .NET. Edge.js takes care of marshalling data between CLR and V8. Edge.js also reconciles threading models of single threaded V8 and multi-threaded CLR, and ensures correct lifetime of objects on V8 and CLR heaps.   

The most powerful aspect of the Node.js scripting capability that Edge.js just enabled is that you can tap onto the many thousands of Node.js modules available both in the Node.js runtime as well as on NPM. For example, you can now create a websocket server in Node.js with a message handler in C#, all running within a single CLR process.  

### Getting started  

Open Visual Studio 2013 and create a new .NET console application. Then add the Edge.js NuGet package to the project using the NuGet Package Manager:  

 ![image](http://lh5.ggpht.com/-ZXMFIAzWqJ8/U3JXsH1GW2I/AAAAAAAAD9Q/0JMWhyZSfOs/image_thumb%25255B1%25255D.png?imgmax=800)   

Now add a using directive for Edge.js:  

{% highlight javascript linenos %}
using EdgeJs;
{% endhighlight %}





And implement the body of the Main method:

{% highlight javascript linenos %}
static void Main(string[] args)
{
    var func = Edge.Func(@"
        return function(data, cb) {
            cb(null, 'Node.js ' + process.version + ' welcomes ' + data);
        };
    ");

    Console.WriteLine(func(".NET").Result);
}
{% endhighlight %}



Compile, run, and enjoy!

 ![image](http://lh3.ggpht.com/-7o6NnQfnLXI/U3JXtA9YEmI/AAAAAAAAD9g/c1DKQNQdJBk/image_thumb%25255B3%25255D.png?imgmax=800) 

### Learn more

Here are more resources to get your started using Edge.js:

[Overview](http://tjanczuk.github.io/edge) 

    
[Get the all-inclusive Edge.js NuGet package](https://www.nuget.org/packages/Edge.js)  
[Scripting Node.js from CLR](https://github.com/tjanczuk/edge#scripting-nodejs-from-clr)  
[What you need](https://github.com/tjanczuk/edge#what-you-need-1)  
[Hello, world](https://github.com/tjanczuk/edge#how-to-nodejs-hello-world)  
[Using built-in Node.js modules](https://github.com/tjanczuk/edge#how-to-use-nodejs-built-in-modules)  
[Using NPM modules](https://github.com/tjanczuk/edge#how-to-use-external-nodejs-modules)  
[Handle Node.js events in C#](https://github.com/tjanczuk/edge#how-to-handle-nodejs-events-in-net)  
[Manage Node.js state from C#](https://github.com/tjanczuk/edge#how-to-expose-nodejs-state-to-net)  
[Script Node.js in ASP.NET](https://github.com/tjanczuk/edge#how-to-use-nodejs-in-aspnet-web-applications)  