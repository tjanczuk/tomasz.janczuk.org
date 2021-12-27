---
tags: ['post']
post_og_image: 'site'
date: '2016-06-07'  
post_title: What is Function as a Service (FaaS)?
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: what-is-function-as-a-service-(faas)?
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---

*Function as a Service* seems to be the new black. But what is it? This brief post will explain it. 

### First, there was a *Function*

Here is the function, saved to the *faas.js* file:

```
javascript
module.exports = function (ctx, cb) {
    cb(null, 'Hello from a Function');
}
```
### Second, we make it a *Service*

Now, we convert the *function* to a *service* with `wt-cli`: 

```

npm install -g wt-cli
wt init
wt create faas.js
```
### Voila, Function as a Service

What comes back is a URL, which is the *Function as a Service*. It may look like this: 

```

https://webtask.it.auth0.com/api/run/tjanczuk/faas
```
Go ahead and call it. It is a service. 

### Wait, what just happened?

We have used the CLI of [Auth0 Webtasks](https://webtask.io) to turn a Node.js function into an HTTP endpoint that can be called from a mobile or HTML5 app, or just with *curl*. Auth0 Webtasks is all about making developer's life simple - all you need is a *function*. Webtasks turn it into *service*. Learn more at [https://webtask.io](https://webtask.io).

### *Function as a Service* vs *Serverless*

How is *Function as a Service* (FaaS) different from *serverless*? It is not. At the core it represents the same trend explained in the [What is serverless?](https://auth0.com/blog/2016/06/09/what-is-serverless/) and on Martin Fowler's blog [Serverless Architectures](http://martinfowler.com/articles/serverless.html). 

The key difference is emotional. The term *server-less* is both inacurate (there are still servers involved), as well as socially controversial, since it suggests looming structural unemployment of swaths of IT personel who make their living by running those servers today. The *Function as a Service (FaaS)* term is clearly less radioactive. Read more about the merit of the trend at [What is serverless](https://auth0.com/blog/2016/06/09/what-is-serverless/). 
}