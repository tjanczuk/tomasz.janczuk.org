---
tags: ['post']
post_og_image: 'site'
date: '2013-01-31'  
post_title: Paste photos from your phone directly into Windows Live Writer
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: paste-photos-from-your-phone-directly-into-windows-live-writer
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




You are writing a blog post using Windows Live Writer. You want to include a picture you took with your phone. The options you have are many but none of them is simple. You can use USB cable, WiFi synchronization, Bluetooth, DropBox, SkyDrive, or Picasa. Or, if you are like me, you can e-mail the pictures to yourself, save to disk, and then include in your blog.   

All this hassle is time consuming and makes you think twice before using pictures from your phone in your blog.   

Enter *[Picture from phone plug-in to Windows Live Writer](http://plugins.live.com/writer/detail/picture-from-phone)*. The plug-in allows you to “paste” pictures from your phone directly into Windows Live Writer. The process takes only a few seconds and does not require any cables, configuration, third party services, or e-mail. Some call it witchcraft, so brace yourself.     

### Witchcraft 101  

Your experience starts with a new Windows Live Writer blog post about, say, your last vacation. You get to the point where you want to include a picture from your phone:  

 ![myvacation1](http://lh4.ggpht.com/-JF__3x29CSA/UQxhDjYxuJI/AAAAAAAADXY/Gp8qks3Xcpo/myvacation1_thumb%25255B2%25255D.png?imgmax=800)   

You go to the “Insert” menu and choose “Picture from phone” (assuming you have [installed the Picture from phone plug-in](http://plugins.live.com/writer/detail/picture-from-phone)):  

 ![myvacation2](http://lh4.ggpht.com/-AaBEhkCns8I/UQxhFwWYFAI/AAAAAAAADXo/OtxCvSfIJss/myvacation2_thumb%25255B1%25255D.png?imgmax=800)   

A window shows up with a QR code to scan. Scanning instructions are also provided for those new to QR codes:  

 ![myvacation3](http://lh6.ggpht.com/-rdgDRgkgi2w/UQxhIG8FIqI/AAAAAAAADX4/TEgY5pvOvZQ/myvacation3_thumb%25255B1%25255D.png?imgmax=800)   

Your phone must have internet connectivity. It is recommended you enable WiFi to speed up the transfer if you can (after all, you will be sending several MB worth of JPEG content; besides speed you also don’t want this to eat into your data plan). As soon as you scan the QR code with your phone, Windows Live Writer detects that (yes, I know, it is magic):  

 ![myvacation4](http://lh3.ggpht.com/-dlrW3cLBT84/UQxhJ69rFyI/AAAAAAAADYI/my2LlnCgOYY/myvacation4_thumb%25255B1%25255D.png?imgmax=800)   

Meanwhile on your phone your QR scanning software redirects you to your mobile browser and opens a web site that allows you to select and upload your picture:  

 ![Screen Shot 2013-02-01 at 12.58.51 PM](http://lh6.ggpht.com/-7IA3St1eZT0/UQxhMnDjAMI/AAAAAAAADYY/R1UqsKyJGMw/Screen%252520Shot%2525202013-02-01%252520at%25252012.58.51%252520PM_thumb%25255B2%25255D.png?imgmax=800)  ![Screen Shot 2013-02-01 at 12.59.16 PM](http://lh5.ggpht.com/-qNxlltljE9g/UQxhO7ap6_I/AAAAAAAADYo/988zm-iKIK0/Screen%252520Shot%2525202013-02-01%252520at%25252012.59.16%252520PM_thumb%25255B2%25255D.png?imgmax=800)  ![Screen Shot 2013-02-01 at 12.59.47 PM](http://lh5.ggpht.com/-YfBUqK3nc00/UQxhQhoC8EI/AAAAAAAADY4/vIdeLBcTbks/Screen%252520Shot%2525202013-02-01%252520at%25252012.59.47%252520PM_thumb%25255B1%25255D.png?imgmax=800)   

  

  

  

And voila! Your picture is now embedded in your blog post:  

 ![myvacation5](http://lh5.ggpht.com/-wVYzk7_dyRc/UQxhSm8-PJI/AAAAAAAADZI/Cqjz34NC7hE/myvacation5_thumb%25255B1%25255D.png?imgmax=800)   

### Witchcraft explained  

To get started, you will need a lizard’s tail, four rotten eggs of the greenback turtle collected on the first full moon in April, and a Windows Azure subscription. The last one is easiest to procure for free [here](http://www.windowsazure.com/en-us/). You also need to understand the basics of [QR codes](http://en.wikipedia.org/wiki/QR_code), [node.js](http://nodejs.org/), [Windows Azure Blob Storage](http://msdn.microsoft.com/en-us/library/windowsazure/dd135733.aspx), and [Windows Azure Web Sites](http://www.windowsazure.com/en-us/develop/nodejs/).   

Here is how you mix it all up. First, when you open the “Picture from phone” plug-in window, a temporary, one-time relay address is registered with a node.js web site hosted in Windows Azure Web Sites. The relay allows messages to be exchanged between devices that know its unique address. The QR code the plug-in displays encodes this one-time relay address. When you scan the QR code with your phone and open the address in the mobile browser, it immediately posts a “hello” message to the relay, which allows the Windows Live Writer on your PC to switch its view and encourage you to choose a picture on the phone. Once you select the picture and upload it to the relay, it is temporarily saved in Windows Azure Blob Storage, and can be referenced there with a unique URL. The URL of the picture is then posted to the relay. When Windows Live Writer receives the message, it fetches the picture from Windows Azure Blob Storage, closes the plug-in window, and includes the picture in the blog post you are editing. The temporary relay address is deleted along with any associated content in the Windows Azure Blob Storage.   

So where does the lizard’s tail come in? I just like to keep it handy in case I need it, so save it for some of the future experiments.   }