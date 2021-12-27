---
tags: ['post']
post_og_image: 'site'
date: '2013-02-27'  
post_title: Access MS SQL from a node.js application using OWIN, in-process CLR
  hosting, .NET, and ADO.NET
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: access-ms-sql-from-nodejs-application
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




*Note: this post is related to the 0.4.0 version of the owin project. The project has since been renamed to edge.js and has seen major improvements. [Visit edge.js](http://tomasz.janczuk.org/2013/03/run-c-and-nodejs-code-in-process-with.html) for the latest.*  

The [owin](https://github.com/tjanczuk/owin) project allows hosting .NET 4.5 code in node.js applications running on Windows. In my previous posts I described how owin can be used to [implement express.js request handlers and connect middleware in .NET](http://tomasz.janczuk.org/2013/02/hosting-net-code-in-nodejs-applications.html) as well as [run CPU-bound computations implemented in .NET within the node.js process](http://tomasz.janczuk.org/2013/02/cpu-bound-workers-for-nodejs.html).  

In this post I will focus on showing how to access a SQL database from a node.js application by hosting CLR in-process and using asynchronous ADO.NET mechanisms, all without writing a single line of C# code.   

### What you need  

You need Windows x64 with node.js 0.8.x x64 installed (the owin module had been developed against node.js [0.8.19](http://nodejs.org/dist/v0.8.19/)). You also need [.NET Framework 4.5](http://www.microsoft.com/en-us/download/details.aspx?id=30653) on the machine.   

The code snippets below assume you have a working MS SQL connection string to [a sample Northwind SQL database](http://www.microsoft.com/en-us/download/details.aspx?id=23654). If the database is deployed on a development machine, the connection string most of the time looks like *Data Source=(local);Initial Catalog=Northwind;Integrated Security=True*.  

### Select  

First install the [owin](https://github.com/tjanczuk/owin) module with  

```

   npm install owin@0.4.0

```


Then in your test.js:

```
var owin = require('owin');  
  
owin.sql("select * from Region", function (error, result) {  
    if (error) throw error;  
    console.log(result);  
});
  

```


Before you run the code, you need to set the connection string to the Northwind SQL database using the OWIN_SQL_CONNECTION_STRING environment variable, e.g:

```

set OWIN_SQL_CONNECTION_STRING=Data Source=(local);Initial Catalog=Northwind;Integrated Security=True
  

```


Now you are ready to run the node.js app with:

```

node test.js
  

```




You will see the following output:

```

C:\projects\owin>;node test.js  
[ [ 'RegionID', 'RegionDescription' ],  
  [ 1, 'Eastern                                           ' ],  
  [ 2, 'Western                                           ' ],  
  [ 3, 'Northern                                          ' ],  
  [ 4, 'Southern                                          ' ] ]
  

```


The result of the query is a JavaScript array. The first element of the array is an array of column names. Subsequent elements of the array are rows representing the results of the SQL select query against the database. 

### Insert

You can insert data into a SQL database with the following code:

```
var owin = require('owin');  
  
owin.sql("insert into Region values (5, 'Pacific Northwest')", function (error, result) {  
    if (error) throw error;  
    console.log(result);  
});
  

```


The result of running this applicaiton indicates how many rows in a SQL table have been affected:

```

C:\projects\owin>node test.js  
1
  

```

Now when you run the same application again, instead of the result the JavaScript callback will receive an error containing a .NET ADO.NET exception indicating a primary key violation in a SQL database, since a row with this ID already exists:   







```

C:\projects\owin>node test.js  
C:\projects\owin\test.js:9  
        if (error) throw error;  
                         ^  
System.AggregateException: One or more errors occurred. ---> System.Data.SqlClient.SqlException: Violation of PRIMARY KE  
Y constraint 'PK_Region'. Cannot insert duplicate key in object 'dbo.Region'.  
The statement has been terminated.
  

```


### Update and delete

Similarly to insert, you can execute SQL update command, followed by a select showing the state of the table:

```
var owin = require('owin');  
  
owin.sql("update Region set RegionDescription='Washington and Oregon' where RegionID=5", function (error, result) {  
    if (error) throw error;  
    console.log(result);  
    owin.sql("select * from Region", function (error, result) {  
        if (error) throw error;  
        console.log(result);  
    });  
});
  

```


The results of running this application are:

```

C:\projects\owin>node test.js  
1  
[ [ 'RegionID', 'RegionDescription' ],  
  [ 1, 'Eastern                                           ' ],  
  [ 2, 'Western                                           ' ],  
  [ 3, 'Northern                                          ' ],  
  [ 4, 'Southern                                          ' ],  
  [ 5, 'Washington and Oregon                             ' ] ]
  

```




The delete SQL command removes rows from the table:

```
var owin = require('owin');  
  
owin.sql("delete Region where RegionID > 4", function (error, result) {  
    if (error) throw error;  
    console.log(result);  
});
  

```




and similarly to insert and update reports the number of rows affected:

```

C:\projects\owin>node test.js  
1
  

```




### How it works

[Owin](https://github.com/tjanczuk/owin) is a native node.js module implemented in C++\CLI that runs on Windows. It hosts CLR runtime within the node.js process. The owin.sql() function invokes ADO.NET asynchronously on a CLR thread within the node.exe process and uses .NET framework to perform the SQL operation. The owin module utilizes the OWIN ([http://owin.org](http://owin.org)) interface to bridge between JavaScript, native, and CLR code. The module takes care of marshaling data between V8 and CLR heaps as well as reconciling threading models. The SQL operation is running asynchronously on CLR threads while the node.js event loop remains unblocked. 

### More

Visit the project page at [https://github.com/tjanczuk/owin](https://github.com/tjanczuk/owin) for the latest bits.

Make sure to check out [implementing express.js request handlers and connect middleware in .NET](http://tomasz.janczuk.org/2013/02/hosting-net-code-in-nodejs-applications.html) as well as [running CPU-bound computations implemented in .NET within the node.js process](http://tomasz.janczuk.org/2013/02/cpu-bound-workers-for-nodejs.html) use cases as well. 

[Feedback](https://github.com/tjanczuk/owin/issues) and pull requests welcome.  