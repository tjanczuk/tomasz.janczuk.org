---
tags: ['post']
post_og_image: 'site'
date: '2011-09-29'  
post_title: Using Event Tracing for Windows to track and diagnose node.js
  applications hosted in IIS/iisnode
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: using-event-tracing-for-windows-to-track-and-diagnose-node.js-applications-hosted-in-iis/iisnode
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




In my recent posts I talked a lot about [iisnode](https://github.com/tjanczuk/iisnode) which allows hosting node.js applications in IIS on Windows. In this post I will describe ways to track and diagnose failures in node.js apps deployed in IIS using Event Tracing for Windows (ETW) just integrated into v0.1.8 of iisnode. ETW is only available on Windows Vista and up and Windows 2008 Server and up.   

### What it is  

Event Tracing for Windows (ETW) is a kernel mode logging feature in Windows with minimal performance overhead. ETW can be turned on or off at any time, even when the application is already running, using a set of tools included in Windows. You can read more about ETW and various ways of using [here](http://msdn.microsoft.com/en-us/magazine/cc163437.aspx).   

### How to use ETW with iisnode  

During installation of iisnode for IIS 7.x or IIS Express 7.x you will get a new script file that helps with capturing ETW traces for iisnode and generating viewable reports from them. The script is located next to iisnode.dll – in the IIS 7.x installation it is at %programfiles%\iisnode.   

Consider calling ‘iisreset’ first to start capturing ETW traces starting from a clean slate. However, you can also perform the procedure when the IIS worker process, iisnode, and node.exe are already running in the system.   

Start by running the etw.bar script from an administrative command prompt:  

 ![image](http://lh3.ggpht.com/-xTEGQaZ9zlk/ToTLEGsmd-I/AAAAAAAAB1U/gQ03LFgVJb0/image_thumb%25255B3%25255D.png?imgmax=800)  

Then run your test scenarios you intend to capture ETW traces for. For example, navigate to the helloworld sample that ships with iisnode:  

 ![image](http://lh5.ggpht.com/-nFAB_D5ZxRY/ToTLE4Fh_cI/AAAAAAAAB1c/MWu-D3f8GS8/image_thumb%25255B6%25255D.png?imgmax=800)  

When you are done with your test scenario, go back to the command line running the etw.bat script and stop capturing ETW traces by pressing a key:  

 ![image](http://lh5.ggpht.com/-aiSpXrEJOvo/ToTLFn-GWLI/AAAAAAAAB1k/KNuAXoXQc70/image_thumb%25255B9%25255D.png?imgmax=800)      

The script created the *.etl file with captured traces that can be post-processed with ETW tools. It also generated an XML report that contains human readable trace log. Press a key one more time and the XML file should open in your default application associated with the *.xml file extension (most likely your browser):  

 ![image](http://lh4.ggpht.com/-Uaa0-s3nU7A/ToTLGf_SZFI/AAAAAAAAB1s/dNsHFlBTnjM/image_thumb%25255B12%25255D.png?imgmax=800)  

### What is being traced  

iisnode currently generates following ETW traces at three verbosity levels:  

1. **Error**  
2. **Information**  
3. **Verbose**  
  

### Under the hood  

If you prefer using ETW tools on your own over invoking the included etw.bat file, you will need the iisnode ETW provider id, which is {1040DFC4-61DB-484A-9530-584B2735F7F7}.  

Enjoy!  }