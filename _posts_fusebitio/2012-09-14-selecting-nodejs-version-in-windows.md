---
tags: ['post']
post_og_image: 'site'
date: '2012-09-14'  
post_title: Selecting the node.js version in Windows Azure Web Sites
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: selecting-the-node.js-version-in-windows-azure-web-sites
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




The latest release of [Windows Azure Web Sites](http://www.windowsazure.com/en-us/develop/nodejs/) adds support for selecting the version of the node.js engine to run your application with. Several node.js versions are currently supported out of the box (0.6.17, 0.6.20, and 0.8.2 as of this writing), with more to come in the future. The node.js version desired by your application is specified in the package.json file using the same mechanisms NPM uses. If no version is specified in the package.json file, Windows Azure Web Sites will choose a default version to run your node.js application with. Using more advanced mechanism, you can also configure your application to use arbitrary node.js versions outside of the pre-installed range, including custom builds.   

### Choosing the node.js version using the package.json file  

To choose a specific version of node.js to run your application with (say 0.6.20), specify that version in the “engines” section of the package.json file:   

```

   {  
    "name": "hellojs",  
    "version": "0.1.0-pre",  
    "engines": {  
        "node": "0.6.20"  
    }  
}
  

```


When you commit and push this change to the Git repository provided by Windows Azure Web Sites, the version of the node.js engine to use is determined based on your choice within package.json:

>
> tjanczuk-air:hellojs tomek$ git push origin master
>
> …
>
> remote: Deploying Web.config to enable Node.js activation.
>
> remote: Selecting nodejs version.
>
> remote: Node.js versions available on the platform are: 0.6.17, 0.6.20, 0.8.2.
>
> remote: Selected node.js version 0.6.20. Use package.json file to choose a different version.
>
> remote: Deployment successful.

Note that within package.json you can use an expression to describe more elaborate version constraints of your application, for example:

```
{  
    "name": "hellojs",  
    "version": "0.1.0-pre",  
    "engines": {  
        "node": "0.6.23 || 0.8.x"  
    }  
}
  

```


In that case, Windows Azure Web Sites will choose the maximum version of node.js installed on the platform at the time of the deployment which satisfies the version constraint specified in package.json. Given the package.json above and node.js versions installed on Windows Azure Web Sites at the time of this writing, the selected node.js version would be 0.8.2:

```

tjanczuk-air:hellojs tomek$ git push origin master 
… 
remote: Selecting nodejs version. 
remote: Node.js versions available on the platform are: 0.6.17, 0.6.20, 0.8.2. 
Selected node.js version 0.8.2. Use package.json file to choose a different version. 
remote: Deployment successful.

```


### The default version of node.js

If the application you deploy to Windows Azure Web Sites does not contain a package.json file, or if the package.json file does not specify the version of node.js to use, Windows Azure Web Sites will use a default node.js version to run your application. The default node.js version in Windows Azure Web Sites will be changing over time (as of this writing the default version is 0.6.20). Given that no backwards compatibility guarantees exist in node.js, you should not rely on this fallback mechanism to ensure continued stability of your application; instead you should always explicitly choose the node.js version to use for your production applications deployed to Windows Azure Web Sites. Fallback to the default node.js version should only be used as a convenient, configuration-less way to get your application started during development. 

### Under the hood and advanced scenarios

Node.js applications in Windows Azure Web Sites are running within [iisnode](https://github.com/tjanczuk/iisnode). The node.js version selection mechanism implemented by Windows Azure Web Sites uses a configuration setting of iisnode to specify the fully qualified file name of the node.exe executable to use for running the application. Specifically, at the time of a new deployment through “git push”, the [nodeProcessCommandLine](https://github.com/tjanczuk/iisnode/blob/master/src/samples/configuration/iisnode.yml#L10-13) setting of the [iisnode.yml](http://tomasz.janczuk.org/2012/05/yaml-configuration-support-in-iisnode.html) configuration file is set to point to one of pre-installed node.exe versions based on the version indicated in package.json. 

If the iisnode.yml file deployed with the application already contains an explicit nodeProcessCommandLine setting, it takes precedence over the node.js version selection logic described above that uses package.json. This allows developers to configure their applications to use arbitrary node.js versions (including custom builds), as long as the node.exe executable to use is deployed as part of the application itself. It is recommended to place the specific node.exe executable to use in the “bin” folder to prevent IIS from serving it as a static file. Next, the nodeProcessCommandLine setting in iisnode.yml file must be configured to specify the fully qualified file name of that executable; in case of Windows Azure Web Sites it should take the following form (the highlighted segment must be replaced with the name of your application): 

```
nodeProcessCommandLine: "C:\\DWASFiles\\Sites\\hellojs\\VirtualDirectory0\\site\\wwwroot\\bin\\node.exe"

```


When the application including node.exe and the iisnode.yml is deployed, the custom version of node.exe will be used to run it: 

```

tjanczuk-air:hellojs tomek$ git push origin master 
… 
remote: Selecting nodejs version. 
remote: The iisnode.yml file explicitly sets nodeProcessCommandLine. Automatic node.js version selection is turned off. 
remote: Deployment successful.

```


This mechanism of selecting a custom node.js version to run your application with had been described in more detail in [a post by Glenn Block](http://codebetter.com/glennblock/2012/06/29/getting-your-azure-web-site-to-use-node-v0-8-1-now/). 

### Bugs? Ideas? Send a pull request

The node.js version selection mechanism employed by Windows Azure Web Sites is part of an open source project [Kudu](https://github.com/projectkudu/kudu); specifically, the version selection logic is implemented [here](https://github.com/projectkudu/kudu/blob/master/Kudu.Core/Scripts/selectNodeVersion.js). Support for the low level mechanism of nodeProcessCommandLine in iisnode.yml is part of the open source [iisnode](https://github.com/tjanczuk/iisnode) project that Windows Azure Web Sites uses to host node.js applications. Both projects are open to contributions.   }