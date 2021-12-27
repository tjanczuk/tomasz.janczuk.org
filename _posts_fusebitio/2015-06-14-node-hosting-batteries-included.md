---
tags: ['post']
post_og_image: 'site'
date: '2015-06-14'  
post_title: Node.js hosting with batteries included
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: node.js-hosting-with-batteries-included
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---

If you are writing a Node.js application today, you have over 160,000 npm modules at your disposal. You declare the ones your application depends on within *package.json*. When the app is deployed, the hosting environment installs these modules before starting the app. The process typically takes between few and few tens of seconds. Only then is your application ready to process requests. 

### What if...

...the hosting environment was pre-provisioned with all the modules your app requires?

In such environment, the provisioning step is much faster, because it no longer requires the dependencies to be installed. The uniform hosting environent can be easily replicated to serve the needs of many different applications, anywhere in the world. The differences between applications are scoped down to application specific code rather than centered around disparate dependency sets. This results in greatly improved agility of deployment: one can run any application anywhere the hosting environment is available with much reduced deployment latency. 

You can try how this concept works in practice on [webtask.io](https://webtask.io), a service provided by Auth0. Auth0 Webtasks allow you to execute Node.js code in a hosting environment pre-provisioned with a large number of Node.js modules. You can also [check which modules are supported on webtask.io](https://tehsis.github.io/webtaskio-canirequire/).

### Napkin math

How viable is this model in practice? While creating a hosting environment with *all* Node.js modules from npm installed is certainly possible, it is hardly practical for several reasons. Many modules on npm are not Node.js modules, or are not meant for use in web workloads. A large number of Node.js modules is either abanondoned, deprecated, or otherwise never meant for general consumption. 

So how should one choose a pragmatic subset of Node.js modules to be pre-installed? One way is to stack rank Node.js modules using the number of downloads of every module in the last 30 days. Download count is a good proxy for module's popularity. 

I've crunched statistical data from npm and here is what I found out: 
![client-server](tomek-blog/2015-06-14/1.png)  
Not surprisingly, a relatively small number of modules on npm contributes to large number of total downloads. What is surprising is the scale of this phenomenon: the most downlodaded 591 modules (that is only 0.4% of the total modules on npm) make up a whooping 80% of the total number of downloads from npm. If you wanted to install modules that contribute to 99% of total downloads from npm, you would still need to install a realtively modest number of 7494 modules (4.9% of total). 

As of this writing, [webtask.io](https://webtask.io) supports all the modules that make up 80 percentile of npm downloads.  

In the next post I will write about how to actually install such a large number of modules without using up too much of disk space. I will also write about supporting several module versions side by side. Stay tuned. 


}