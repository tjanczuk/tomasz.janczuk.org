---
tags: ['post']
post_og_image: 'site'
date: '2016-07-15'  
post_title: Serverless MySQL with Auth0 Webtasks
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: serverless-mysql-with-auth0-webtasks
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---

In this post, we will take a parmeterized T-SQL query:

```
sql
select * from crime where crimedescr like ? limit 100
```
and use [Auth0 Webtasks](https://webtask.io) to turn it into an HTTP endpoint:

```
bash
> wt create crime.sql ...
```
The endpoint can we used to execute parameterized T-SQL queries against a MySQL database with public crime records in Sacramento: 

<img src="tomek-blog/2016-07-15/1.png" alt="T-SQL Auth0 Webtask">

Intrigued? Read on.

### Serverless and Domain Specific Languages

The new [serverless](https://auth0.com/blog/2016/06/09/what-is-serverless/) trend focuses software development on *functions* as units of application logic. These functions are typically implemented in high level programming languages and frameworks like Node.js, Java, Python, or .NET. This offers a lot of flexibility, but sometimes using a domain specific language (DSL) like T-SQL or a custom programming model allows for a much better development experience. 

> What if you could take a T-SQL script and expose it directly over HTTP without having to write boilerplate Node.js, Python, or Java code around it?

In this post I will show how [Auth0 Webtasks](https://webtask.io) allow you to use the *webtask compiler* mechanism to enable creating HTTP endpoints implemented in domain specific languages and further streamline serverless development experience.

### Crime in Sacramento

Before we can use T-SQL we must have some exciting data to query in a MySQL database. For the sake of this exercise, I created a free MySQL database on [freemysqlhosting.net](https://www.freemysqlhosting.net/) and populated it with public crime records for Sacramento which I got in CSV format [here](https://support.spatialkey.com/spatialkey-sample-csv-data/).

<img src="tomek-blog/2016-07-15/0.png" alt="Serverless crime in Sacramento">

Bottom line is you need the hostname of the MySQL server, database name, and user name and password to connect to it. 

### Access MySQL through Function-as-a-Service (FaaS) with Auth0 Webtasks

Let's say we want to allow querying our crime database to match for patterns on the crime description field (*crimedescr* above), and expose it over HTTP such that the search pattern can be specified through URL query parameters. Traditionally such functionality would be implemented in any of the high level languages. In case of Auth0 Webtasks, the following Node.js code could be written to accomplish the task: 

```
javascript
var mysql = require('mysql');

module.exports = function (ctx, cb) {

    // Create MySQL connection on first invocation
    if (!global.connection) {
        // Validate that required MySQL connection parameters 
        // were specified at webtask creation
        var secrets = ['HOST','DB','USER','PASSWORD'];
        for (var i = 0; i < secrets.length; i++) 
            if (!ctx.secrets[secrets[i]]) 
                return cb(new Error('You must specify the ' 
                    + secrets[i] 
                    + ' secret when creating the webtask.'));

        // Create MySQL connection and cache it in-memory 
        // for use by later webtask requests
        global.connection = mysql.createConnection({
            host     : ctx.secrets.HOST,
            user     : ctx.secrets.DB,
            password : ctx.secrets.PASSWORD,
            database : ctx.secrets.USER
        });
        global.connection.connect();
    }

    // Execute parameterized MySQL query setting parameter 
    // value based on the value of the URL query 
    // parameter `q`. Webtask will return MySQL error or
    // a JSON array of matching rows.
    global.connection.query(
        'select * from crime where crimedescr like ? limit 100', 
        ctx.query.q, 
        cb);
};
```
> Notice that the business logic of this code boils down to a single line of a parameterized T-SQL script. The rest of it is boilerplate code in Node.js.

The unexciting boilerplate is reposible for validating parameters, establishing MySQL connection, and running the query. Notice also how the parameterized T-SQL query is passed values from the URL query paramater `q` stored in `ctx.query.q` at runtime.

Turning this Node.js module into an HTTP endpoint using Auth0 Webtasks is simple with the *wt-cli* command line tool (assuming the code above is stored in *crime.js*): 

```
bash
npm i -g wt-cli
wt init

wt create crime.js \
  -s HOST={mysql_host} \
  -s DB={mysql_db} \
  -s USER={mysql_user} \
  -s PASSWORD={mysql_password}
```
Notice how the key/value pairs specified using the `-s` option are provided at runtime to the running code using the `ctx.secrets` hash. This unique mechanisms allows you to easily provision your webtask code with secrets without much ceremony around managing how they are stored and protected, and without having to store them directly in your code. You can read more about this Auth0 Webtask security model [here](https://webtask.io/docs/how). 

When the command runs, a URL is retuned that can be used to invoke the Node.js function over HTTP. Using the `q` URL query parameter you can limit the results by providing a T-SQL pattern to match against the crime description field. Try the following URL in the browser to find information about burglaries (crime code 459):

[https://webtask.it.auth0.com/api/run/tjanczuk/crime?q=459%](https://webtask.it.auth0.com/api/run/tjanczuk/crime?q=459%)

The response will contain a JSON array with matching MySQL records:

<img src="tomek-blog/2016-07-15/1.png" alt="T-SQL Auth0 Webtask">

While the development experience of creating this HTTP endpoint is already streamlined compared to many more traditional ways of deploying web applications, there is still room for improvement. 

### Webtask compilers and domain specific languages

If you look at the Node.js code we had to write to execute the T-SQL query against the MySQL database, it is clear the majority of it is a generic boilerplate functionality around the core business logic captured in the T-SQL script. 

> Webtask compilers allow webtasks to be implemented in domain specific languages like T-SQL by providing a mechanism to externalize the code that adapts between custom and webtask programming models.

Given a webtask script using a domain specific language (e.g. T-SQL), an Auth0 Webtask can be created from it by associating the script with a *webtask compiler* at the time of webtask creation. The compiler will be invoked at runtime to translate the custom programming model to one of the programming models natively supported by webtasks. The result is cached and as such performance impact is minimal. 

### Access MySQL through a T-SQL Auth0 Webtask

To put it all together, consider the essential business logic of the previous example implemented as a single line of parameterized T-SQL script stored in *crime.sql* file: 

```
sql
select * from crime where crimedescr like ? limit 100
```
This is the code you care about as a developer, and Auth0 Webtasks with *webtask compilers* allow you to focus on it rather than surrounding boilerplate. 

Similarly to the previous example, you can turn this code into an Auth0 Webtask HTTP endpoint using *wt-cli* command line tool:

```
bash
wt create crime.sql --name crime \
  --meta wt-compiler=http://bit.ly/29ZDXVt \
  -s HOST={mysql_host} \
  -s DB={mysql_db} \
  -s USER={mysql_user} \
  -s PASSWORD={mysql_password}
```
The key element of this command is the `--meta` parameter, which specifies the *webtask compiler* to associate with the webtask. Webtask compilers can we specified as URLs that resolve to code in Node.js. That code executes in the webtask environment to perform the adaptation of the custom webtask script to one of the supported webtask programming models. Read more about the webtask compiler model and how to implement one [here](https://webtask.io/docs/webtask-compilers). 

> Webtask compilers allow for reuse of the logic that enables domain specific languages in Auth0 Webtasks. 

Webtask compilers introduce separation of concerns between the logic that enables a custom programming model and the logic implemented *in* that custom programming model. Compilers can be reused across webtasks and therefore further improve the webtask development experience. If you inspect the code of the T-SQL webtask compiler at [http://bit.ly/29ZDXVt](http://bit.ly/29ZDXVt) you will notice it indeed captures the very boilerplate glue code in Node.js we had to use in the first implementation of our webtask. 

Now with the reusable compiler in place, webtask development can focus on the business logic in T-SQL:

<img src="tomek-blog/2016-07-15/2.png" alt="T-SQL Auth0 Webtask in Webtask Editor Widget">

### What else can you do with webtask compilers

Webtask compilers can be used to enable webtask development in a variety of languages and frameworks, as long as they can be transpiled to Node.js. We have seen support for T-SQL above. You can use webtask compilers to directly support Express programming model. You can enable webtask authoring in Jade, Ejs, or any other templating language. You can support custom Node.js programming models specific to your application domain. 

You can take it as far as supporting implementation of webtasks in C#, by using the [Edge.js](https://github.com/tjanczuk/edge) module to execute CLR code in-process with Node.js: 

<img src="tomek-blog/2016-07-15/3.png" alt="C# Auth0 Webtask in Webtask Editor Widget using Edge.js">

You can read more about using the webtask compilers [here](https://webtask.io/docs/model#webtask-compilers), and about implementing your own [here](https://webtask.io/docs/webtask-compilers). Enjoy!
