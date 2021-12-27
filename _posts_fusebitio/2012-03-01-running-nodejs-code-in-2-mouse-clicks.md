---
tags: ['post']
post_og_image: 'site'
date: '2012-03-01'  
post_title: Running node.js code in 2 mouse clicks using haiku-http
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: running-node.js-code-in-2-mouse-clicks-using-haiku-http
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




If you are reading this, you have obviously clicked already once to get to this post. So you only have 1 click left – use it on one of the sample links below.   

Then you can [experiment more with haiku-http](http://tjanczuk.github.com/haiku-http/), read about [haiku-http vision and design](http://tomasz.janczuk.org/2012/02/sub-process-multi-tenant-runtime-for.html), or [get the code on GitHub](https://github.com/tjanczuk/haiku-http).  

### Hello, world  

Here is the ‘Hello, world’ rite of passage:  

[http://haiku.cloudapp.net/?x-haiku-handler=https://raw.github.com/tjanczuk/haiku-http/master/samples/haikus/hello.js](http://haiku.cloudapp.net/?x-haiku-handler=https://raw.github.com/tjanczuk/haiku-http/master/samples/haikus/hello.js)  

```
   res.writeHead(200)  
res.end('Hello, world!\n')

```


### War and peace

How many times do the words “war” and “peace” appear on [http://reuters.com](http://reuters.com) today?

War: 
    
[http://haiku.cloudapp.net/?x-haiku-handler=https://raw.github.com/tjanczuk/haiku-http/master/samples/haikus/request.js&word=war](http://haiku.cloudapp.net/?x-haiku-handler=https://raw.github.com/tjanczuk/haiku-http/master/samples/haikus/request.js&word=war)

Peace: 
    
[http://haiku.cloudapp.net/?x-haiku-handler=https://raw.github.com/tjanczuk/haiku-http/master/samples/haikus/request.js&word=peace](http://haiku.cloudapp.net/?x-haiku-handler=https://raw.github.com/tjanczuk/haiku-http/master/samples/haikus/request.js&word=peace) 

```
var query = require('url').parse(req.url, true).query  
var word = query.word || 'the'  
var request = require('request')  
request('http://www.reuters.com', function (error, response, body) {  
    if (error || response.statusCode !== 200) {  
        res.writeHead(500)  
        res.end('Unexpected error getting http://reuters.com.\n')  
    }  
    else {  
        var count = 0, index = 0  
        while (0 !== (index = (body.indexOf(word, index) + 1)))  
            count++  
        res.writeHead(200)  
        res.end('Number of times the word "' + word + '" occurs on http://reuters.com is: ' + count + '\n')  
    }  
})

```


### Fetch data from MongoDB

Return documents from MongoDB that match search criteria.

All documents: 
    
[http://haiku.cloudapp.net/?x-haiku-handler=https://raw.github.com/tjanczuk/haiku-http/master/samples/haikus/mongo.js](http://haiku.cloudapp.net/?x-haiku-handler=https://raw.github.com/tjanczuk/haiku-http/master/samples/haikus/mongo.js)

Only documents for ‘app1.com’ host: 
    
[http://haiku.cloudapp.net/?x-haiku-handler=https://raw.github.com/tjanczuk/haiku-http/master/samples/haikus/mongo.js&host=app1.com](http://haiku.cloudapp.net/?x-haiku-handler=https://raw.github.com/tjanczuk/haiku-http/master/samples/haikus/mongo.js&host=app1.com)

```
var query = require('url').parse(req.url, true).query  
var mongoUrl = query['db'] || 'mongodb://arr:arr@staff.mongohq.com:10024/arr'  
var filter = query['host'] ? { hosts: query['host'] } : {}  
  
require('mongodb').connect(mongoUrl, function (err, db) {  
    if (notError(err))  
        db.collection('apps', function (err, apps) {  
            if (notError(err))  
                apps.find(filter).toArray(function (err, docs) {  
                    if (notError(err)) {  
                        res.writeHead(200)  
                        res.end(JSON.stringify(docs))  
                    }  
                })  
        })  
})  
  
function notError(err) {  
    if (err) {  
        res.writeHead(500)  
        res.end(err)  
    }  
    return !err  
}
  

```




### Time for more

[Experiment more with haiku-http](http://tjanczuk.github.com/haiku-http/), read about [haiku-http vision and design](http://tomasz.janczuk.org/2012/02/sub-process-multi-tenant-runtime-for.html), or [get the code on GitHub](https://github.com/tjanczuk/haiku-http). 

Comments welcome!  }