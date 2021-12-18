---
post_title: When a JavaScript function is not a function
date: 2012-03-15
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: when-a-javascript-function-is-not-a-function
post_date_in_url: true
post_og_image: site
post_excerpt: Tomek on Software - shaken, not stirred
tags:
  - post
---




One would think that a function is a function is a function. Right? Wrong.   

Consider this node.js code:  

{% highlight javascript linenos %}
   var f = function () {}  
console.log('f instanceof Function in the main context: ' + (f instanceof Function))  
  
require('vm').runInNewContext(  
    "console.log('f instanceof Function in child context: ' + (f instanceof Function))",  
    { console: console, f: f })  

  

{% endhighlight %}



Turns out that a function in the main V8 context is not a function in the child V8 context, at least according to JavaScript's instanceof:

{% highlight text linenos %}
f instanceof Function in the main context: true  
f instanceof Function in child context: false
  

{% endhighlight %}



The reason for this can be easily explained after reading (and probably re-reading a few times) [the great write-up on prototypical inheritance](http://joost.zeekat.nl/constructors-considered-mildly-confusing.html). The f function is created in the main context and has main context's Function object in its prototype chain. When instanceof is run in the child context against the f function created in the main context, it fails to find the child context's Function object in f's prototype chain. Knowing that, the problem can be fixed by sharing main context's Function object with the child context using the global object:

{% highlight javascript linenos %}
var f = function () {}  
console.log('f instanceof Function in the main context: ' + (f instanceof Function))  
  
require('vm').runInNewContext(  
    "console.log('f instanceof Function in child context: ' + (f instanceof Function))",  
    { console: console, f: f, Function: Function })  

  

{% endhighlight %}



Which leads to the expected:

{% highlight text linenos %}
f instanceof Function in the main context: true  
f instanceof Function in child context: true
  

{% endhighlight %}



However, sharing objects between V8 contexts probably defeats the purpose of using separate V8 contexts in the first place, at least for some applications. After all, V8 contexts are meant to isolate in memory state. So a safe alternative that does not require sharing the Function object is using 

{% highlight javascript linenos %}
typeof f === 'function'
  

{% endhighlight %}



check instead of the original

{% highlight javascript linenos %}
f instanceof Function
  

{% endhighlight %}



End of trivia.  }