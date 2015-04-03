---
layout: post
title: Run C# and node.js code in-process with no compilation
date: '2013-03-12T14:32:00.001-07:00'
author: Tomasz Janczuk
tags:
- JavaScript
- edgejs
- edge.js
- HTTP
- development
- node.js
- .net
- OWIN
- edge
- V8
- CLR
modified_time: '2013-03-16T09:34:46.278-07:00'
thumbnail: http://lh5.ggpht.com/-_7HdUzmOMos/UUP-YrbL9QI/AAAAAAAADcA/vvJqCXJrOlI/s72-c/image_thumb%25255B2%25255D.png?imgmax=800
blogger_id: tag:blogger.com,1999:blog-2987032012497124857.post-4313724663069057913
blogger_orig_url: http://tomasz.janczuk.org/2013/03/run-c-and-nodejs-code-in-process-with.html
---




The [edge.js](http://tjanczuk.github.com/edge) project allows running .NET and node.js code in-process. With the edge@0.7.0 release,  you can now embed C# code in directly in the node.js application, without the hassle of projects, compilation, and DLLs. Edge.js will compile the C# code automatically in-memory before running it.  

*Note: the edge.js project was previously called “owin”. If you know it by that name, [this](https://github.com/tjanczuk/owin) explains why it was renamed.*  

 ![image](http://lh5.ggpht.com/-_7HdUzmOMos/UUP-YrbL9QI/AAAAAAAADcA/vvJqCXJrOlI/image_thumb%25255B2%25255D.png?imgmax=800)   

Edge.js provides a prescriptive, asynchronous model for calling .NET code from node.js and node.js code from .NET. Edge.js module takes care of marshaling data between V8 and CLR and reconciling the threading models. And with edge@0.7.0 you no longer have to deal with C# compilation and CLR DLLs as edge.js compiles C# sources for you on the fly.   

Everything you need to get started is covered at [http://tjanczuk.github.com/edge](http://tjanczuk.github.com/edge). Poll requests and [feedback](https://github.com/tjanczuk/edge/issues) are welcome.  

For specific topics check out previous posts:  

[Run node.js and .NET code in-process](http://tomasz.janczuk.org/2013/03/run-nodejs-and-net-code-in-process.html)       
[Access MS SQL from node.js using OWIN](http://tomasz.janczuk.org/2013/02/access-ms-sql-from-nodejs-application.html)       
[Implement CPU-bound computations on CLR threads within node.js process](http://tomasz.janczuk.org/2013/02/cpu-bound-workers-for-nodejs.html)       
[Hosting .NET code in node.js process using OWIN](http://tomasz.janczuk.org/2013/02/hosting-net-code-in-nodejs-applications.html)  