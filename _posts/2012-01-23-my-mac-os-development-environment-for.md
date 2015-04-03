---
layout: post
title: My Mac OS development environment for node.js
date: '2012-01-23T14:29:00.001-08:00'
author: Tomasz Janczuk
tags:
- whiteboard
- GitHub
- Mac OS
- development
- node.js
- gissues
- express
- web
- scrum
modified_time: '2012-01-23T14:29:39.232-08:00'
thumbnail: http://lh6.ggpht.com/-69-vM3jHbi8/Tx3fUd1XhvI/AAAAAAAAB64/Htr6X9W4GDU/s72-c/Screen-Shot-2012-01-23-at-12.03.24-P.png?imgmax=800
blogger_id: tag:blogger.com,1999:blog-2987032012497124857.post-1285919960647943157
blogger_orig_url: http://tomasz.janczuk.org/2012/01/my-mac-os-development-environment-for.html
---




I have recently released [http://gissues.com](http://gissues.com), a whiteboard for [GitHub](http://github.com) issues. It was the first web application I developed entirely on Mac OS. As such there were several choices I needed to make regarding technologies to use as well as development environment and tools. This post describes in some detail these choices.   

#### What does gissues do?  

The [gissues](http://gissues.com) application provides a whiteboard-like user interface for GitHub issues. It allows you to organize issues in four columns (backlog, not started, in progress, done). You can move issues around and reorder them with intuitive drag & drop gestures. The whiteboard-like interface is especially useful for driving GitHub projects using SCRUM.   

 ![http://gissues.com, whiteboard for GitHub issues](http://lh6.ggpht.com/-69-vM3jHbi8/Tx3fUd1XhvI/AAAAAAAAB64/Htr6X9W4GDU/Screen-Shot-2012-01-23-at-12.03.24-P.png?imgmax=800)  

The gissues application is backed directly by the GitHub issue repository and stores whiteboard metadata within the GitHub issue itself. It helps avoid duplication of data between an issue tracking system and another system used to support the SCRUM or SCRUM-like process.  

#### The technology  

The gissues application consists of three components: browser client, web server, and the backend.   

The web server is built using [node.js](http://nodejs.org) with a [Express](http://expressjs.com) framework and [EJS](https://github.com/visionmedia/ejs) rendering engine. The web server logic is very thin – its main purpose is to broker OAuth authentication between the browser client and the GitHub backend. It involves a few asynchronous HTTP calls to [GitHub v3 APIs](http://developer.github.com/v3/) which node.js allows to be implemented efficiently with minimal coding.   

The HTML views the web server generates are extensively using AJAX to communicate with the GitHub v3 APIs for issue manipulation. The calls are authenticated using the OAuth token obtained from GitHub when the user logs in. The web server is not involved in brokering these calls, which further improves scalability of the web server. The client code uses [jQuery](http://jquery.com) for general DOM manipulation and AJAX, and [jQuery UI](http://jqueryui.com) for the drag & drop gestures. Since I am no CSS expert, I benefited a lot from using [Twitter’s Bootstrap](http://twitter.github.com/bootstrap/) styles and layout.   

The gissues application has no backend of its own – all application metadata is stored directly within the GitHub issue description, e.g.   

>
> @gissues:{"order":78.515625,"status":"notstarted"}  

This simplifies the design and helps achieve full fidelity between the state of the GitHub issue repository and the gissues whiteboard. For example, creating a new issue in GitHub automatically adds this issue to gissues’s backlog; closing an issue in GitHub makes this issue disappear from gissues.   

#### The development environment  

There are probably as many ways to set up an ergonomic development environments as there are devs. For someone who has spent the better part of last decade writing code on Windows, below is the set of tools and practices I found convenient.   

I am using [Sublime Text 2](http://www.sublimetext.com/) for my text editor. It offers most of the features I care about, and some I only learned I care about when I started using them. I particularly appreciate the “Search everything” feature, the maximization of the use of the screen real estate, and the mini map of the file content, and the ability to quickly open an entire directory structure for editing. Syntax highlighting is nowadays a bread & butter of code editors, and Sublime does a great job there as well.   

Since my web server was using node.js, I found it indispensible to use [supervisor](https://github.com/isaacs/node-supervisor) in the course of development. Supervisor will restart the server every time one of the specified set of watched files changes. This way one can make a quick change in the text editor and switch directly to the browser window to refresh it without worrying about restarting the node server. Supervisor accepts a set of file extensions of files to watch; in my particular project I configure it listen to *.js files (client side JS files, node.js server code), *.css files (my styles), and *.ejs files (EJS view templates that I render into HTML through the node.js server:  

>
> supervisor -e "js\|ejs\|css" server.js  

In terms of the browser choice, I use [Google Chrome](https://www.google.com/chrome/). It has all the development tools I care about built in (most of them are quickly becoming a commodity across all browser brands and models). One can debug client side JS code, inspect CSS, correlate between the rendered view of HTML and the actual HTML elements, and inspect network calls. That was just enough for my development and debugging needs.   

One interesting aspect of developing of a web app is the fact that one often needs to hardcode URLs into the application code that are not yet publicly available. For example, when obtaining an OAuth token for accessing GitHub issues using GtiHub v3 APIs one needs to provide a URL that GitHub will redirect to upon successful authorization. Once the application is live, that URL is going to be the publicly available URL, e.g. [http://gissues.com](http://gissues.com) in my case. The question was, how do I make GitHub redirect to a service that is running on my development machine during development? One way was to change the URL of the OAuth application registered in GitHub, but that would not longer work after the initial deployment of the application. Another one was to register two OAuth applications in GitHub, one that I would use for production, and second for development. But that would require me to modify my application code when going from development to production. The least intrusive approach I landed on was to simply change the /private/etc/hosts file on my Mac to resolve DNS name of what would become my production domain name to localhost by adding one line:  

>
> 127.0.0.1     gissues.com  

This approach allows me to leave my application code as well as GitHub OAuth application configuration with production settings, and localize the development mode behavior to my own machine. After deployment, in order to access the actual application on the internet, all I need to do is to comment out the above line in /private/etc/hosts.   

#### The deployment  

The [http://gissues.com](http://gissues.com) app is deployed on a Micro Ubuntu instance in [Rackspace](http://www.rackspace.com/) (around $11/month plus the cost of bandwidth, which is minimal given the heavy reliance of the application on AJAX).   

Deploying to a raw VM for the first time I needed to decide on a convenient mechanism to do so. I played around with some fancy SFTP clients but they appeared to be just too much to handle. It crossed by mind to just install git client on the box and pull whenever I needed to refresh the bits, but I finally settled on a less sophisticated approach: I just tar the artifacts and sftp them up to my server:  

>
> tar czf gissues.tgz gissues/
>
> sftp gissues.com
>
> …
>
> put gissues.tgz  

Then I ssh to the box and untar:  

>
> tar xzf gissues.tgz  

The last issue I needed to address was running the node.js server on the Ubuntu box in a way that provides a degree of reliability. I looked at the forever module that ensures a node.js process is restarted when it crashes, but I finally settled on Ubuntu’s native [upstart](http://upstart.ubuntu.com/). Upstart allows one to start a process on system startup and restart it when it crashes, with a level of rapid failure protection. Plus it takes a minute to configure and get running using great instructions from [Kevin van Zonneveld](http://kevin.vanzonneveld.net/techblog/article/run_nodejs_as_a_service_on_ubuntu_karmic/).   