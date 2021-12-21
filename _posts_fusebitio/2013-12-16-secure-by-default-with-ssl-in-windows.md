---
tags: ['post']
post_og_image: 'site'
date: '2013-12-16'  
post_title: Secure by default with SSL in Windows Azure Web Sites
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: secure-by-default-with-ssl-in-windows-azure-web-sites
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




Windows Azure Web Sites (WAWS) allow your web applications to be exposed over HTTP and HTTPS. After you [configure your Azure web site with an SSL certificate](http://www.windowsazure.com/en-us/develop/net/common-tasks/enable-ssl-web-site/), by default your endpoints will be reachable over both HTTP and HTTPS. For some apps you may want to prevent the use of HTTP and require callers to always use HTTPS. How do you do this within an web app deployed to Windows Azure Web Sites?  

A simple practice to promote your site’s security is to detect if a call is made over HTTP and redirect the caller to the corresponding HTTPS endpoint instead. Implementation of this mechanism in Windows Azure Web Sites requires understanding of how HTTPS is implemented in WAWS.   

When a caller is making an HTTPS request to your site, the request first arrives at WAWS router based on the ARR technology. This is where the SSL connection from the client is terminated. After performing SSL handshake on your site’s behalf, ARR forwards the client request to the actual server running your application code *over unsecured HTTP connection.* (This is OK, since this traffic is internal to an Azure data center). However, it raises the question of how your web application can detect if the original client call was made over HTTP or HTTPS? It turns out the ARR is attaching a special HTTP request header to every request that arrives over HTTPS. The name of the header is *x-arr-ssl* and its value contains information about the SSL server certificate that was used to secure the TCP connection between the client and the ARR.   

 ![image](http://lh4.ggpht.com/-7hqbK_-GnMk/Uq9uNervUwI/AAAAAAAAD3g/hGmEk_TbMSU/image_thumb%25255B1%25255D.png?imgmax=800)   

An application deployed to Windows Azure Web Sites can detect presence of the *x-arr-ssl* header in deciding whether to redirect client’s call to an HTTPS endpoint, or continue processing it. This approach can be implemented with any web application technology. The example below shows a simple Connect middleware for Node.js applications deployed to WAWS that allow redirecting all traffic to HTTPS endpoints:  

{% highlight javascript linenos %}
   function ensureHttps(redirect) {  
    return function (req, res, next) {  
        if (req.headers['x-arr-ssl']) {  
            next();  
        }  
        else if (redirect) {  
            res.redirect('https://' + req.host + req.url);  
        }  
        else {  
            res.send(404);  
        }  
    }  
}
  

{% endhighlight %}



The middleware will detect HTTPS request and continue processing them. If an HTTP request arrives, it can be either redirected to a corresponding HTTPS endpoint, or flat out rejected with an HTTP 404 response, depending how the middleware is configured. As a rule of thumb, if the request contains sensitive information and it was received over plain HTTP, it should be rejected. Otherwise, it is OK to redirect it to a corresponding HTTPS endpoint. Here is how you can use this middleware in configuring endpoints of an Express application:

{% highlight javascript linenos %}
app.post('/',  
    ensureHttps(true), // This is a home page, redirect HTTP to HTTPS  
    routes.home);  
  
app.get('/account',  
    ensureHttps(false), // Authenticated endpoint, reject HTTP with a 404  
    authenticate(),  
    routes.account);

{% endhighlight %}



You can see the redirection from HTTP to HTTPS in action when you navigate to [http://mobilechapters.com](http://mobilechapters.com). 

 ![image](http://lh4.ggpht.com/-ohTEbsw55ss/Uq9uOM2UadI/AAAAAAAAD3w/eBux_Jp-wIo/image_thumb%25255B8%25255D.png?imgmax=800) 

Enjoy!  }