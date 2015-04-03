---
layout: post
title: Developing node.js applications in WebMatrix and testing in IIS Express on
  Windows
date: '2011-08-30T18:34:00.001-07:00'
author: Tomasz Janczuk
tags:
- JavaScript
- HTTP
- development
- node.js
- IIS
- iisnode
- Windows
- webmatrix
- web
- browser
modified_time: '2011-08-31T10:24:34.534-07:00'
thumbnail: http://lh6.ggpht.com/-KK_0hT3xFjQ/Tl2PpGZ0w1I/AAAAAAAAB0c/5faYD7601Iw/s72-c/image_thumb%25255B1%25255D.png?imgmax=800
blogger_id: tag:blogger.com,1999:blog-2987032012497124857.post-1028564601823711892
blogger_orig_url: http://tomasz.janczuk.org/2011/08/developing-nodejs-applications-in.html
---




In my last few posts I introduced the [iisnode](https://github.com/tjanczuk/iisnode) project and talked about specific aspects of hosting node.js applications in IIS 7.x on Windows:  

[Hosting node.js applications in IIS on Windows](http://tomasz.janczuk.org/2011/08/hosting-nodejs-applications-in-iis-on.html)   
[Integration with the URL rewrite module](http://tomasz.janczuk.org/2011/08/using-url-rewriting-with-nodejs.html)         
[Running node.js express applications](http://tomasz.janczuk.org/2011/08/hosting-express-nodejs-applications-in.html)   

In this article I am showing how to use the WebMatrix development environment to develop node.js applications and test them in IIS Express using the same iisnode module. WebMatrix is a free, lightweight, Windows based development environment for web application development using a variety of web technologies.   

### Setting it up  

First you need to [download and install WebMatrix](http://www.microsoft.com/web/webmatrix/) (free). The download will also include IIS Express, a lightweight version of IIS specifically designed for development purposes.   

Next you need to [download a recent x86 build of iisnode](https://github.com/tjanczuk/iisnode/archives/master). Unzip it somewhere on disk.     

**Note** if you are running 64 bit Windows, you still need to download the x86 build of iisnode, since IIS Express only ships in 32 bit version.    

Now install iisnode by calling the installation script from the command line *with administrative privileges*:   

On a 32 bit Windows system:  

{% highlight text linenos %}
     install_iisexpress.bat

{% endhighlight %}



On a 64 bit Windows system:

{% highlight text linenos %}
%systemroot%\syswow64\cmd.exe /C install_iisexpress.bat

{% endhighlight %}



If everything goes well, you should see successful installation confirmation:

 ![image](http://lh6.ggpht.com/-KK_0hT3xFjQ/Tl2PpGZ0w1I/AAAAAAAAB0c/5faYD7601Iw/image_thumb%25255B1%25255D.png?imgmax=800)

Take note of the samples directory to use it from WebMatrix in the next step.

### Start up WebMatrix

Start up WebMatrix. When prompted, choose “Site from folder”:

 ![image](http://lh6.ggpht.com/-3VWZ7_4a18E/Tl2Pp3wpoqI/AAAAAAAAB0k/Vkrhv5t7rSI/image_thumb%25255B3%25255D.png?imgmax=800)

Enter the samples path that was shown in the last step of iisnode installation:

 ![image](http://lh3.ggpht.com/-53JZjEEoSWQ/Tl2Pq9he-1I/AAAAAAAAB0s/vwEsqix_mjg/image_thumb%25255B5%25255D.png?imgmax=800)

You can now explore and modify all node.js samples that came with iisnode installation in WebMatrix:

 ![image](http://lh3.ggpht.com/-tniBWla_URQ/Tl2PtcGw-NI/AAAAAAAAB00/u3P3xtr2HUw/image_thumb%25255B8%25255D.png?imgmax=800)

Right click on index.htm and choose “Launch in browser”. IIS Express will host the samples web site and you will be able to access the node.js sample endpoints exposed from IIS Express: 

 ![image](http://lh4.ggpht.com/-yD6dJDDhrnI/Tl2Ptzpnq6I/AAAAAAAAB08/7zLrBicLEjI/image_thumb%25255B10%25255D.png?imgmax=800)

### What did just happen?

WebMatrix with IIS Express gives you a lightweight, free development stack for node.js applications on Windows. IIS Express allows you to exercise the same set of features iisnode exposes in a regular IIS installation, including URL rewriting module. 

### So where can I get iisnode again?

Everything you need to get started is at [https://github.com/tjanczuk/iisnode](https://github.com/tjanczuk/iisnode). Make sure to check out [the samples](https://github.com/tjanczuk/iisnode/tree/master/src/samples). Feedback welcome!  