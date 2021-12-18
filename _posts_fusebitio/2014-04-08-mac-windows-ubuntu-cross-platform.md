---
post_title: Mac, Windows, Ubuntu cross-platform development
date: 2014-04-08
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: mac,-windows,-ubuntu-cross-platform-development
post_date_in_url: true
post_og_image: site
post_excerpt: Tomek on Software - shaken, not stirred
tags:
  - post
---




“Grand[m\|p]a” (have to be careful with [pronouns](http://www.joyent.com/blog/the-power-of-a-pronoun) this year), “what did you use to write cross-platform software back in 2014? No, really… do tell… Wow, amazing… It is, *[like](https://www.villagetheatre.org/everett/The-Tutor.php)*, they really did not have […]?”  

This post describes a cross platform development setup that proved efficient for *me* in the course of several cross platform projects (most notably [Edge.js](http://tjanczuk.github.io/edge/#/)). As anything in technology, this is point in time: it has as much practical value for my contemporary engineers as it going to have entertainment value for my daughter.   

Cutting to the chase:  

 ![image](http://lh6.ggpht.com/-E0oxmYndrrs/U0OvKXhdVgI/AAAAAAAAD6E/86EtcxUOA7s/image_thumb%25255B3%25255D.png?imgmax=800)   

  

I am using a MacBook Pro 13” (my shoulders are getting too old to drag along the 15”) with SDD 512GB (my ears are too old to listen to the HDD hum) and 16GB RAM (my nerves are too strung to wait for Windows to do its thing).   

I am running Windows 8.1 and Ubuntu 12.04 in a VM using VMWare Fusion. Including the MacOS host, this captures most of my x-platform target.   

I share the home folder on the MacBook Pro to both Ubuntu and Windows WMs. This is the quickest way to share files across the host and guest OSes.  

I use Git[Hub] for sharing public artifacts between my fellow developers and my own development machines.   

I use OneDrive for sharing private artifacts between my development machines. I suppose you could use DropBox, but I am psychologically biased towards OneDrive.   

I use Sublime Text for a uniform code editing experience across platforms. It is sublime. And text. Plus, Commodore is *so much better* than Atari. Bottom line it works x-platform and has all these fancy colors, unlike vi. Could not resist.   

I use Visual Studio for those infrequent tasks that require Windows specific work. It also comes really handy when profiling code. Turns out some of the code that is slow on Windows is *also* slow on *nix and MacOS. Yes, at the end of the day everything boils down to E=mc^2.   

I use Windows Live Writer to write this post.   }