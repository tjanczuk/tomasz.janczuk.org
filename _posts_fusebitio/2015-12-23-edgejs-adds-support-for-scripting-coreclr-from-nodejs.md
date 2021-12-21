---
tags: ['post']
post_og_image: 'site'
date: '2015-12-23'  
post_title: Edge.js adds support for scripting CoreCLR from Node.js
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: edge.js-adds-support-for-scripting-coreclr-from-node.js
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---

The [Edge.js project](https://github.com/tjanczuk/edge) allows you to script CLR from Node.js in-process on Windows, OSX, and Linux.

```javascript
var edge = require('edge');

var helloWorld = edge.func(function () {/*
    async (input) => { 
        return ".NET Welcomes " + input.ToString(); 
    }
*/});

helloWorld('JavaScript', function (error, result) {
    if (error) throw error;
    console.log(result);
});
```

<img src="/assets/images/blog/tomek_blog/2015-12-23/0.png" class="tj-img-diagram-100" alt="Scripting CoreCLR from Node.js using Edge.js on Windows, OSX, and Linux">

Until now, Edge.js allowed scripting *desktop CLR* on Windows, and *Mono* on OSX and Linux. 

Today, thanks the fantastic effort of [Luke Stratman](http://careers.stackoverflow.com/lstratman)...

> **Edge.js 5.0 allows you to script CoreCLR from Node.js on Windows, OSX, and Linux**

With this major release, Edge.js provides a uniform, cross-platform mechanism that supports in-process Node/CoreCLR interoperability and facilitates building hybrid, single process applications. 

<img src="/assets/images/blog/tomek_blog/2015-12-23/2.jpg" class="tj-img-diagram-75" alt="Edge.js interoperabilty model between Node.js and CoreCLR">

Microsoft's CoreCLR project provides an open source implementation of the CLR technology that works uniformly on Linux, OSX, and Windows. Prior to CoreCLR, applications using Edge.js had to utilize desktop CLR on Windows and Mono on OSX or Linux, which complicated cross-platform development and introduced subtle (and some less subtle) behavioral differences. 

### Getting started with CoreCLR and Edge.js

You can find platform specific instructions for installing Edge.js with CoreCLR support in the [Edge.js Getting Started](https://github.com/tjanczuk/edge#contents) section of the documentation. 

By far the easiest way to experiment with scripting C# from Node.js via Edge.js is to use the `tjanczuk/edgejs:5.0.0` Docker image: 

```
> docker run -it tjanczuk/edgejs:5.0.0
> cd samples
> node 101_hello_lambda.js
.NET welcomes Node.js
```

The docker image is based on Debian Jessie and comes with Node.js 4.2.3 x64, Mono 4.2.1 x64, CoreCLR 1.0.0 RC1 x64, and Edge.js pre-installed and ready to use. 

In environments with both Mono and CoreCLR installed (like this Docker image), Edge.js will by default use Mono. You need to opt-in to using CoreCLR by setting the `EDGE_USE_CORECLR` environment variable: 

```
> docker run -it tjanczuk/edgejs:5.0.0

> cat > hello.js <<EOF
var hello = require('edge').func(function () {/*
  async (x) => {
    return "CoreCLR welcomes " + x.ToString();
  }
*/});
hello('Node.js', function (error, result) {
  if (error) throw error;
  console.log(result);
});
EOF

> EDGE_USE_CORECLR=1 node hello.js
CoreCLR welcomes Node.js
```

### You can also script Node.js from CLR

In addition to scripting CLR from Node.js, Edge.js also allows the opposite: scripting Node.js from CLR. This is currently only supported on Windows using desktop CLR, but support for CoreCLR is coming! Here is the complete picture of what you can do with Edge.js: 

<img src="/assets/images/blog/tomek_blog/2015-12-23/1.png" class="tj-img-diagram-100" alt="Edge.js support matrix for scripting CLR, CoreCLR, Mono from Node.js and Node.js from CLR on Windows, OSX, and Linux">

Script away!

}