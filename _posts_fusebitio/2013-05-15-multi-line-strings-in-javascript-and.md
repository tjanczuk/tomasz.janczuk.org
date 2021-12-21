---
tags: ['post']
post_og_image: 'site'
date: '2013-05-15'  
post_title: Multi-line strings in JavaScript and Node.js
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: multi-line-strings-in-javascript-and-node.js
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
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

Enjoy!  }