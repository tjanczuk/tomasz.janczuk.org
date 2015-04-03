---
layout: post
title: iisnode v0.1.10 has shipped
date: '2011-11-22T15:25:00.001-08:00'
author: Tomasz Janczuk
tags:
- JavaScript
- HTTP
- node.js
- IIS
- iisnode
- node-inspector
- debugging
- web
modified_time: '2011-11-22T15:25:20.771-08:00'
blogger_id: tag:blogger.com,1999:blog-2987032012497124857.post-6752044088527053857
blogger_orig_url: http://tomasz.janczuk.org/2011/11/iisnode-project-allows-hosting-node.html
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