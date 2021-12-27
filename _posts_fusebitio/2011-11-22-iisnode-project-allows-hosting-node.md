---
tags: ['post']
post_og_image: 'site'
date: '2011-11-22'  
post_title: iisnode v0.1.10 has shipped
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: iisnode-project-allows-hosting-node
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




The [iisnode project](https://github.com/tjanczuk/iisnode) allows hosting node.js applications in IIS on Windows. Some of the [benefits of hosting node.js in IIS are outlined here](https://github.com/tjanczuk/iisnode/wiki). Version 0.1.10 of iisnode has just shipped and includes the following improvements:  

1. Added performance and stress test framework with a first basic test. The framework requires the [WCAT](http://www.iis.net/community/default.aspx?tabid=34&g=6&i=1466) load testing tool for IIS.  
2. Node-inspector artifacts have been split out into a separate DLL to reduce the working set in production scenarios. Node-inspector is now loaded into the IIS worker process on demand only when debugging functionality is used.  
3. Added support for NODE_ENV. iisnode will propagate the value of the iisnode/@node_env configuration setting from web.config to an environment variable of the node.exe processes it spawns. This enables frameworks like express or connect to vary their configuration depending on the deployment environment.  
4. Added support for node.js applications that begin sending HTTP response before receiving the HTTP request in full.  
5. Added support for disabling the debugger with the iisnode/@debuggerEnabled configuration setting.  
6. Fixed handling of HTTP requests with Expect: 100-continue request headers.  
7. Fixed installer issue that prevented iisnode installation if the SP1 of Visual Studio C++ 2010 Redistributable Package was present on the machine instead of the RTM version. [Anthony Vanderhoorn]  
8. The [MSI installers for iisnode v0.1.10](https://github.com/tjanczuk/iisnode/downloads) are now signed for a more streamlined installation experience.  
9. Other assorted bug fixes that increase stability and correctness.  
  

This version of iisnode requires [node.js version 0.6.2](http://nodejs.org/#download) or greater. Enjoy!  