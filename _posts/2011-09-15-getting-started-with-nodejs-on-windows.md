---
layout: post
title: Getting started with node.js on Windows
date: '2011-09-15T17:26:00.001-07:00'
author: Tomasz Janczuk
tags:
- JavaScript
- development
- node.js
- IIS
- iisnode
- Windows
- webmatrix
- express
- web
modified_time: '2011-09-15T17:26:53.682-07:00'
blogger_id: tag:blogger.com,1999:blog-2987032012497124857.post-1683238117549830966
blogger_orig_url: http://tomasz.janczuk.org/2011/09/getting-started-with-nodejs-on-windows.html
---




Here are a few quick steps to set up a node.js development environment on Windows:  

1. Install WebMatrix: [http://www.microsoft.com/web/webmatrix/](http://www.microsoft.com/web/webmatrix/)  
2. Install node.js: [https://github.com/tjanczuk/node/downloads](https://github.com/tjanczuk/node/downloads)  
3. Install iisnode for IIS Express (choose x86 even on 64 bit systems): [https://github.com/tjanczuk/iisnode/downloads](https://github.com/tjanczuk/iisnode/downloads)  
4. Install Steve’s node.js templates for WebMatrix: [https://github.com/SteveSanderson/Node.js-Site-Templates-for-WebMatrix/downloads](https://github.com/SteveSanderson/Node.js-Site-Templates-for-WebMatrix/downloads)  
5. Open WebMatrix, choose “Site from folder”, enter %localappdata%\iisnode\www, start the site, and play with the iisnode samples, or  
6. Use node.js templates to get started quickly with an Express application or a skeleton Hello World  
  

For more information and howtos check out my recent posts and visit [http://github.com/tjanczuk/iisnode](http://github.com/tjanczuk/iisnode).   

I hope you like it and I am looking for your comments and suggestions. Please report any issues at [https://github.com/tjanczuk/iisnode/issues](https://github.com/tjanczuk/iisnode/issues).   