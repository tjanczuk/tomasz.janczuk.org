---
layout: post
title: Multi-line strings in JavaScript and Node.js
date: '2013-05-15T11:19:00.001-07:00'
author: Tomasz Janczuk
tags:
- JavaScript
- edgejs
- node.js
- edge
modified_time: '2013-05-15T11:19:10.064-07:00'
thumbnail: http://lh3.ggpht.com/-Sh6hlijb6QA/UZPRnPSDY5I/AAAAAAAADd8/aVNuHMP6buc/s72-c/fs.ps.py.cs.js_thumb%25255B2%25255D.png?imgmax=800
blogger_id: tag:blogger.com,1999:blog-2987032012497124857.post-3851111564512703940
blogger_orig_url: http://tomasz.janczuk.org/2013/05/multi-line-strings-in-javascript-and.html
---




When writing Node.js or JavaScript applications, you sometimes need to embed multi-line strings in your code. It may be a snippet of HTML, a fragment of textual template, a piece of XML (remember XML?), or code in another programming language.   

JavaScript has no built-in way of representing multi-line strings. If you need to embed a longer non-JavaScript text in your application the natural options are limited to concatenating several one-line JavaScript strings or using external files. Unless, of course, you use a little known trick:  

{% highlight javascript linenos %}
   var html = (function () {/*  
  <!DOCTYPE html>  
  <html>  
    <body>  
      <h1>Hello, world!</h1>  
    </body>  
  </html>          
*/}).toString().match(/[^]*\/\*([^]*)\*\/\}$/)[1];
  

{% endhighlight %}



What happens here? An anonymous function is created with a function body consisting only of a multi-line comment. The comment itself is the very text you want to embed in your application. The function is then serialized to a string using *toString()*. Interestingly, the call preserves the function signature along with the function body and the comments within. Last, a regular expression is run over the serialized form of the function to extract the comment hidden inside. The end result assigned to the *html* variable contains the HTML content within the comment. 

That pattern can be applied to a variety of types of multi-line text. For example, I am using the pattern liberally in the [Edge.js](http://tjanczuk.github.io/edge) project to embed (and later run) fragments of C#, F#, Python, or PowerShell code in a Node.js application:

 ![fs.ps.py.cs.js](http://lh3.ggpht.com/-Sh6hlijb6QA/UZPRnPSDY5I/AAAAAAAADd8/aVNuHMP6buc/fs.ps.py.cs.js_thumb%25255B2%25255D.png?imgmax=800) 

Enjoy!  