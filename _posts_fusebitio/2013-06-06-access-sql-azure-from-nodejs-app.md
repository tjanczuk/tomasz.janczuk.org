---
post_title: Access SQL Azure from a Node.js app deployed to Windows Azure Web Sites
date: 2013-06-06
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: access-sql-azure-from-a-node.js-app-deployed-to-windows-azure-web-sites
post_date_in_url: true
post_og_image: site
post_excerpt: Tomek on Software - shaken, not stirred
tags:
  - post
---




In this post I will show how you can access SQL Azure data from an Express.js Node.js application deployed to Windows Azure Web Sites, and expose that data over HTTP in 31 lines of Node.js and T-SQL code.   

This post demonstrates collaboration between several technologies: in-process interop between Node.js and ADO.NET using [Edge.js](http://tjanczuk.github.io/edge) and [Edge-sql](https://github.com/tjanczuk/edge#how-to-script-t-sql-in-a-nodejs-application), SQL Azure, Windows Azure Web Sites, and ADO.NET.   

### Prerequisites  

You need a Windows Azure subscription. You can [get started for free here](http://www.windowsazure.com/en-us/pricing/free-trial/) if you don’t have one.   

You need to know how to deploy a simple Node.js *hello, world* application to Widows Azure Web Sites. If you have not done it before, [here is a walkthrough](http://www.windowsazure.com/en-us/develop/nodejs/tutorials/create-a-website-(mac)/). You don’t need a Windows development machine to follow through, it works from Mac, *nix, and Windows just as well.   

### Create SQL Azure database  

If you don’t have a SQL Azure database you could experiment with, create one now through the [Windows Azure Management portal](https://manage.windowsazure.com). Once created, go to the management dashboard of the database and open the T-SQL console:  

 ![image](http://lh4.ggpht.com/-XH4A5QIzVIQ/UbA8RPUL8NI/AAAAAAAADgQ/KylXHCNJUr0/image_thumb%25255B5%25255D.png?imgmax=800)   

In the T-SQL console, create a simple *Products* table and populate it with three entries:  

 ![image](http://lh3.ggpht.com/-mH-TPUKTvUI/UbA8R9cnGBI/AAAAAAAADgc/BNZnsoFwmrE/image_thumb%25255B4%25255D.png?imgmax=800)   

Your SQL Azure database should now have three rows in the *Products* table:  

 ![edgewaws-products](http://lh6.ggpht.com/-NREU7kIZD6U/UbA8SjBHB5I/AAAAAAAADgw/AvpJ2erSx8A/edgewaws-products_thumb%25255B1%25255D.png?imgmax=800)   

The last step is to capture the ADO.NET connection string to your SQL Azure database. Again this can be done from the Windows Azure Management dashboard for SQL:  

 ![image](http://lh4.ggpht.com/-vgxqevygLv0/UbA8TRaOd1I/AAAAAAAADhA/IFxahMka4AY/image_thumb%25255B7%25255D.png?imgmax=800)   

Make sure to capture the ADO.NET connection string, and replace the *{your_password_here}* placeholder with the actual SQL Server password you specified when creating the SQL Server:   

 ![image](http://lh4.ggpht.com/-qvPm187vyP0/UbA8UNaPW3I/AAAAAAAADhQ/d6jXBI2e5qg/image_thumb%25255B9%25255D.png?imgmax=800)   

### The Windows Azure Web Site  

I am going to assume you have a simple Node.js application already running in the Windows Azure Web Sites, and that you know how to update it. If you don’t, [follow the walkthrough](http://www.windowsazure.com/en-us/develop/nodejs/tutorials/create-a-website-(mac)/).   

Before you create an Express.js application that exposes SQL Azure data over HTTP, you will need to pass the connection string to that application using a Windows Azure Web Sites mechanism of *app settings.* App settings are key/value pairs configured on the management portal of a Windows Azure Web Site. In the context of a Node.js application, these settings are propagated as environment variables to the Node.js process running your application. As such, they provide a convenient mechanism for passing secrets, like database connections strings, to the application.   

Go to the management dashboard of your Windows Azure Web Site Node.js application, navigate to the *Configuration* tab, and set the *EDGE_SQL_CONNECTION_STRING* app setting to the value of the ADO.NET connection string of the SQL Azure database you captured in the previous step:  

 ![edgewaws-connectionstring](http://lh6.ggpht.com/-JHL-f66qlps/UbA8U1R63YI/AAAAAAAADhc/IJ4q6Cg53Dk/edgewaws-connectionstring_thumb%25255B1%25255D.png?imgmax=800)   

After you enter the value, make sure to click on *Save* below to reset the Node.js application and enable it to access the new app setting via an environment variable.  

### The Node.js application  

The Node.js application is going to use Express.js framework to expose two HTTP endpoints for accessing the data in the *Products* table of the SQL Azure database:  

* One endpoint, */products*, will return a JSON array containing all products from the *Products* table,  
* Another endpoint, */products/<id>*, will return JSON representing the product with *ProductID* equal to *<id>*.  
  

The application is going to use the [Edge.js](http://tjanczuk.github.io/edge) module for Node.js to enable in-process introp with .NET. It is also going to use [Edge-sql](https://github.com/tjanczuk/edge#how-to-script-t-sql-in-a-nodejs-application) extension of Edge.js which allows scripting T-SQL inside of a Node.js application using .NET Framework’s ADO.NET client. This is the package.json of the application which declares the dependencies:  

 ![image](http://lh4.ggpht.com/-3JUO8Q2WJuY/UbA8VbeuQrI/AAAAAAAADhw/uAaq-BJ5IEA/image_thumb%25255B11%25255D.png?imgmax=800)   

The actual Express.js application is using Edge.js and Edge-sql to communicate with the SQL Azure database and expose the results as JSON:  

 ![image](http://lh5.ggpht.com/-x2mn3dopfC4/UbA8WIhY-lI/AAAAAAAADiA/jPzmVyu3wk0/image_thumb%25255B13%25255D.png?imgmax=800)   

Lines 4-7 use Edge.js to create the *getProducts* JavaScript function. The function uses Edge-sql extension of Edge.js to provide in-process introp with ADO.NET functionality that executes the specified T-SQL query over a SQL database. The connection string to the SQL database is provided through the *EDGE_SQL_CONNECTION_STRING* environment variable set in the Node.js process as a result of configuring the Windows Azure Web Site *app setting* in the previous step.   

Lines 9-13 create a similar *getProductById* JavaScript function. This function is created over a parameterized T-SQL query. The actual value of the *@productId* parameter is provided based on a HTTP url segment of the corresponding HTTP GET call in lines 24-25.   

Lines 17-22 expose the */products* endpoint which uses the *getProducts* function to obtain a list of all products in the *Products* table of the SQL Azure database identified with the *EDGE_SQL_CONNECTION_STRING* connection string. The list of products is returned to the caller of the HTTP API as a JSON array:  

 ![edgewaws-productsall](http://lh3.ggpht.com/-gl-E7n20VrE/UbA8W53_3RI/AAAAAAAADiQ/OenGXJ8XM_0/edgewaws-productsall_thumb%25255B1%25255D.png?imgmax=800)   

Lines 24-29 expose the */products/<id>* endpoint which uses the *getProductById* function parameterized with the product ID value passed as a URL segment of the HTTP GET request to obtain information about that particular product. The information is returned to the caller of the HTTP API as JSON:  

 ![edgewaws-productsone](http://lh5.ggpht.com/-G-eXud3V6bw/UbA8Xi2K6zI/AAAAAAAADig/Id2aLZfZeBk/edgewaws-productsone_thumb%25255B1%25255D.png?imgmax=800)   

### What’s next  

The [Edge.js](http://tjanczuk.github.io/edge) module for Node.js allows in-process interop beween Node.js and .NET code. The [Edge-sql](https://github.com/tjanczuk/edge#how-to-script-t-sql-in-a-nodejs-application) extension of Edge.js enables executing T-SQL scripts embedded within a Node.js application using asynchronous ADO.NET running in-process with Node.js code. Therefore it enables access to MS SQL databases, including but not limited to SQL Azure databases. The Edge-sql extension currently supports the four basic CRUD operations: select, insert, update, and delete.   

Enjoy. Collaboration welcome: [https://github.com/tjanczuk.edge](https://github.com/tjanczuk.edge).   }