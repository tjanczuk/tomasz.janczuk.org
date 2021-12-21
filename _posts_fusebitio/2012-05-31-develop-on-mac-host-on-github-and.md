---
tags: ['post']
post_og_image: 'site'
date: '2012-05-31'  
post_title: Develop on Mac, host on GitHub, and deploy to Windows Azure with git-azure
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: develop-on-mac,-host-on-github,-and-deploy-to-windows-azure-with-git-azure
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




If you are like most node.js developers, you develop your code on a Mac and host it on GitHub. With the [git-azure](https://github.com/tjanczuk/git-azure) project I started recently, you can now deploy that code to Windows Azure directly from your development environment.   

### What is git-azure?  

Git-azure is a tool and runtime environment that allows deploying multiple node.js applications into Windows Azure Worker Role from MacOS using Git in seconds.   

A good way to understand how git-azure can help you deploy and manage node.js applications is to watch the 7 minute video below.    


<iframe height="315" src="http://www.youtube.com/embed/X634KGNw0pM" frameborder="0" width="560" allowfullscreen="allowfullscreen"></iframe>
      
  

Git-azure consists of three components: a git-azure runtime, a command line tool integrated into git toolchain, and your own Git repository (likely on GitHub).  

The git-azure runtime is a pre-packaged Windows Azure service that runs HTTP and WebSocket reverse proxy on an instance of Windows Azure Worker Role. The proxy is associated with your Git repository that contains one or more node.js applications. Incoming HTTP and WebSocket requests are routed to individual applications following a set of convention based or explicit routing rules.  

The git-azure command line tool is an extension of the git toolchain and accessible with the git azure command. It allows you to perform a one-time deployment of git-azure runtime associated with your Git repository to Windows Azure, after which adding, modifying, and configurting applications is performend with regular git push commands and take seconds. The git azure tool also helps with scaffolding appliations, configuring routing and SSL, and access to Windows Azure Blob Storage.  

### Motivation and direction  

I have started the [git-azure](https://github.com/tjanczuk/git-azure) project as a platform for experimenting with a variety of ideas related to streamlining development and deployment of applications to Windows Azure. Current focus is primarily on node.js applications.  

Key design decisions at this point were derived from two observations:  

* most node.js developers currently use MacOS to develop node.js applications,  
* [vast majority of node.js developers use Git to manage their projects](http://twtpoll.com/loydbz).  
  

The initial release of git-azure (v0.2.0) supports the following scenarios and functionality:  

* develop on Mac, host code on GitHub, and deploy from Mac to Windows Azure Worker Role,  
* tight integration of the client side tool with the git toolchain,  
* run multiple self-hosted node.js applications behind an HTTP reverse proxy on a single VM running in Windows Azure,  
* after initial provisioning of the VM, add and modify applications in seconds using git push and a post-receive hook,  
* support for HTTP and WebSocket traffic,  
* SSL support out of the box,  
* customization of SSL credentials per host name using Server Name Identification (SNI),  
* administrative access to the VM using RAS (currently requires Windows client).  
  

In the nearest future, I am planning to look at the following:  

* real time access to application logs and diagnostic information,  
* SSH access to the VM for administration,  
* scale out to multiple VM instances and customization of VM size  
  

### How to learn more and engage  

Do check out the walkthrough at the project site at [https://github.com/tjanczuk/git-azure](https://github.com/tjanczuk/git-azure) for hands-on experience with the git-azure tool and runtime as well as access to the code.   

Please share your thoughts or ideas by [filing an issue](https://github.com/tjanczuk/git-azure/issues), or consider contributing to the project. I do take contributions.   

### Special thanks  

Special thanks to [Glenn Block](http://twitter.com/#!/gblock), the first user of git-azure, for valuable feedback and ideas, as well as helping spread the news about git-azure across several node.js events he spoke at recently in Europe.   

Also a big thank you to [Matias Woloski](http://twitter.com/#!/woloski) who was kind enough to talk about git-azure during JSConf.AR 2012.   }