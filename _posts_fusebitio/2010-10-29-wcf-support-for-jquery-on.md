---
post_title: WCF support for jQuery on wcf.codeplex.com
date: 2010-10-29
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: wcf-support-for-jquery-on-wcf.codeplex.com
post_date_in_url: true
post_og_image: site
post_excerpt: Tomek on Software - shaken, not stirred
tags:
  - post
---




Today we have shipped the first release of WCF Web APIs on [wcf.codeplex.com](http://wcf.codeplex.com/). WCF Web APIs is a set of open source components that make it easy to develop WCF web services that target web and HTTP clients, including JavaScript Ajax applications running in a browser.   


 ![image[1]](http://lh3.ggpht.com/_NUp_nWDyyvI/TMr9kcCLuiI/AAAAAAAABnA/nmtZcx6fKFk/image14.png?imgmax=800)  


One of the components is [WCF support for jQuery](http://wcf.codeplex.com/wikipage?title=WCF%20jQuery). It provides great Visual Studio experience for creating WCF services that are optimized for consumption from JavaScript clients running in a browser, in particular jQuery. This includes better support for the JSON format as well as application/x-www-form-urlencoded data. Submitting an HTML form to a WCF service with an Ajax call from jQuery has never been easier. Let’s have a look at the feature.  


### Installation

You must have Visual Studio 2010 to use the bits. Downloading and installing the [WCF Support for jQuery 10.10.27](http://wcf.codeplex.com/releases/view/54705) release from wcf.codeplex.com will install Visual Studio templates, as well as copy reference files and samples to %ProgramFiles%\Microsoft SDKs\WCF Support for jQuery.   


### Visual Studio project template

Create a new project and choose the “WCF jQuery Service Application” template in the “Web” group:  


 ![image](http://lh4.ggpht.com/_NUp_nWDyyvI/TMr9l9ISaEI/AAAAAAAABnI/-G9JhoQ7vcA/image_thumb6.png?imgmax=800)  


The template creates a web application project that contains a WCF service (WcfJQueryService.cs) and a sample HTML page that demonstrates how to invoke that service with an Ajax call using jQuery (default.htm):  


 ![image](http://lh4.ggpht.com/_NUp_nWDyyvI/TMr9mgs32cI/AAAAAAAABnQ/x9RWYqwJi6E/image_thumb5.png?imgmax=800)  


The project references include two assemblies that have been installed as part of the WCF Support for jQuery release (you get [get the source code](http://wcf.codeplex.com/SourceControl/list/changesets) of these assemblies from the wcf.codeplex.com site). System.Runtime.Serialization.Json.dll contains the JsonValue component which facilitates operating on JSON data in an untyped manner (JsonValue is to JSON what XElement is to XML). JsonValue supports serialization and deserialization to the JSON format. It [has already shipped in Silverlight before](http://msdn.microsoft.com/en-us/library/system.json(VS.95).aspx), now we are porting it with some enhancements to .NET Framework.  


 ![image](http://lh5.ggpht.com/_NUp_nWDyyvI/TMr9na8Ns7I/AAAAAAAABnY/JryqFenTe4E/image_thumb8.png?imgmax=800)  


The Microsoft.ServiceModel.Web.jQuery.dll contains a number of WCF components that integrate JsonValue with the WCF HTTP programming model, and add support for processing JSON and application/x-www-form-urlencoded data, which are the two most commonly used data formats for Ajax calls in JavaScript applications.   


The WCF Support for jQuery package also includes an item template that allows a jQuery-enabled WCF service to be added to an existing web application project.   


### From jQuery client to WCF service

Let’s have a look at the WCF service to see what the programming experience is:

{% highlight csharp linenos %}
[WebInvoke(UriTemplate = "", Method = "POST")]
public JsonValue Post(JsonObject body)
{
    dynamic item = body;
    item.id = items.Count;
    items.Add(item);
    return items[items.Count - 1];
}

{% endhighlight %}



The method above is extending the WCF HTTP programming model available since .NET Framework 3.5 SP1 with support for JsonValue as the method parameter and return value type. If JSON or application/x-www-form-urlencoded data arrives in the body of the POST request, it will be deserialized into the JsonObject instance passed to the Post method. The JsonValue instance the method returns will be serialized into JSON format on the HTTP response.   


The default.html file shows how this method can be called from JavaScript, in this case using the Ajax call from the jQuery framework:

{% highlight javascript linenos %}
var person = {
    Name: "John Doe",
    Age: 21
};
$.post("./Service/", person, function (result) {
    //...
});

{% endhighlight %}



In line 1-4 a JavaScript object ‘person’ is created. Line 5 initiates a POST HTTP request to the WCF service identified with a relative ‘./Service/’ URL, and passes the ‘person’ instance to be sent to the service. By default, jQuery serializes the JavaScript object using the application/x-www-form-urlencoded format. The HTTP request looks like the one below, with line 4 containing the serialized ‘person’ instance:

{% highlight text linenos %}
POST http://127.0.0.1:8326/Service/ HTTP/1.1
Content-Type: application/x-www-form-urlencoded
 
Name=John+Doe&Age=21

{% endhighlight %}



The interesting aspect of WCF support for application/url-form-encoded is that JavaScript applications can use that format to serialize not only simple lists of key/value pairs traditionally associated with an HTML form submission, but also arbitrarily complex JavaScript objects. JsonValue supports deserialization of data from application/x-www-form-urlencoded format [following the jQuery’s $.param() serialization conventions](http://benalman.com/news/2009/12/jquery-14-param-demystified/). To see this format in action, let’s send a more complex object to the server by modifying the ‘person’ instance to:

{% highlight javascript linenos %}
var person = {
    Name: "John Doe",
    Age: 21,
    Children: [
        {
            Name: "Jessica",
            BestToy: "Fish"
        },
        {
            Name: "Jeff",
            BestToy: "Donkey"
        }
    ]
};

{% endhighlight %}



The corresponding HTTP request with this data is shown below, with line 4 containing the complex ‘person’ structure serialized into an application/x-www-form-urlencoded format using the $.param() convention from jQuery:

{% highlight text linenos %}
POST http://127.0.0.1:8326/Service/ HTTP/1.1
Content-Type: application/x-www-form-urlencoded
 
Name=John+Doe&Age=21&Children%5B0%5D%5BName%5D=Jessica&Children%5B0%5D%5BBestToy%5D=Fish&Children%5B1%5D%5BName%5D=Jeff&Children%5B1%5D%5BBestToy%5D=Donkey

{% endhighlight %}



Although jQuery Ajax calls use application/x-www-form-urlencoded format by default, it is very easy to serialize the request payload in JSON format instead. Using Douglas Crockford’s [json2](http://json.org/json2.js) serializer (line 4), one can make a jQuery call like this:

{% highlight javascript linenos %}
$.ajax({
    type: "POST",
    url: "./Service/",
    data: JSON.stringify(person),
    contentType: "application/json"
});

{% endhighlight %}



Which results in the following HTTP request:

{% highlight text linenos %}
POST http://127.0.0.1:8326/Service/ HTTP/1.1
Content-Type: application/json
 
{"Name":"John Doe","Age":21,"Children":[{"Name":"Jessica","BestToy":"Fish"},{"Name":"Jeff","BestToy":"Donkey"}]}

{% endhighlight %}



A great feature of the JsonValue programming model in WCF is that regardless of the client’s choice of JSON or application/x-www-form-urlencoded as the serialization format for the request, WCF will deserialize and normalize the data into an instance of JsonValue. Given that, the application code may stay decoupled from the protocol.   


So how does one access the data the client sent from the WCF service? JsonValue offers dictionary-like programming model. For example, to determine the favorite toy of the second child of the person, one could write the following code:

{% highlight csharp linenos %}
[WebInvoke(UriTemplate = "", Method = "POST")]
public JsonValue Post(JsonObject body)
{
    string favoriteToyOfSecondChild = (string)body["Children"][1]["BestToy"];
    return favoriteToyOfSecondChild;
}

{% endhighlight %}



If the return type of a WCF method is JsonValue, the response content type is always application/json. Note that in line 5 above we are actually returning a string instance, but an implicit cast exists between a number of primitive data types and JsonValue. In this case the specific HTTP response would look as follows:

{% highlight text linenos %}
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
 
"Donkey"

{% endhighlight %}



Surely enough, donkey is the favorite toy of John’s second child. 

### JsonValue 201: dynamic properties

JsonValue supports a few more interesting constructs that facilitate operating on untyped data.   


First of all, there is support for dynamic properties, which is more than just a syntactic improvement over array indexers. The two lines below extract the same value from a JsonObject:

{% highlight csharp linenos %}
JsonObject body;
string favoriteToyOfSecondChild1 = (string)body["Children"][1]["BestToy"];
string favoriteToyOfSecondChild2 = (string)body.AsDynamic().Children[1].BestToy;

{% endhighlight %}



The notation in line 2 is closer to the code one would expect to write to navigate an object in JavaScript, and some people find it more natural than using indexers in line 1.   


Given that the data received from the client has not been validated against a specific schema of a CLR type, one cannot be sure the JsonObject really contains the data the server application expects, or whether the object even has the correct structure. For example, a malicious client could try to send an arbitrary string in the “Age” field instead of a number, or could completely omit the Children array. In that case, the indexer APIs (line 1 above) would throw an exception as soon as the code attempts to access a non-existing property (which is consistent with the behavior of indexers in the .NET Framework in general). One would therefore have to write a lot of validation code before accessing the data in the JsonValue object using indexers, especially if the data hierarchy is deep.  


The dynamic properties of JsonValue offer a much more convenient validation experience. One can “dot into” any level of a complex object without any exceptions being thrown, only to perform a test of the value’s existence and CLR type match at the very end, with a very [Fluent-like](http://en.wikipedia.org/wiki/Fluent_interface) experience.

{% highlight csharp linenos %}
JsonObject body;
string aValue;
if (!body.AsDynamic().Children[7].BestToy.TryReadAs<string>(out aValue))
{
    aValue = "None"; // assume a default value
}

{% endhighlight %}



You can imagine how this programming model is reducing the amount of code you need to write when navigating deep data hierarchies:

{% highlight csharp linenos %}
body.AsDynamic().I.May.Exist.Or.Maybe.Not.TryReadAs<string>(out aValue)

{% endhighlight %}



### JsonValue 301: LINQ to JSON

Here is another useful feature JsonValue offers: LINQ to JSON. Using the ‘person’ object introduced above, let’s assume you are writing a method that is supposed to return the favorite toys of those children of the person whose name begin with ‘J’ (does this sound like a test from your SQL class?). Here is the code you could write:

{% highlight csharp linenos %}
JsonObject body;
string[] favoriteToys =
    (from child in (JsonValue)body.AsDynamic().Children
     where child.Value.AsDynamic().Name.ReadAs<string>(string.Empty).StartsWith("J")
     select child.Value.AsDynamic().BestToy.ReadAs<string>("No favorite toy")).ToArray();

{% endhighlight %}



There are several interesting properties of the code above:

1. You don’t have to worry whether the ‘Children’ property exists on the ‘body’ JsonObject (line 3), or whether it is a JSON array. If it does not, the LINQ expression will just not return any matches. No exception is going to be thrown.  
2. You don’t have to worry whether the ‘Name’ property exists on a child object (line 4). Instead, you can provide a default name in the ReadAs method that will be returned in the absence of the property.  
3. Similarly, in case a child whose name starts with ‘J’ has been found, you don’t have to worry whether the child has a ‘BestToy’ property (line 5) – if she does not, you can just fall back to returning a ‘No favorite toy’ string.  


In other words, writing compact and robust LINQ expressions against JSON data with unknown structure is easy. 

### Read more

I have only touched on the key features of the WCF support for jQuery release. Find out more in the [dedicated documentation section](http://wcf.codeplex.com/documentation) on wcf.codeplex.com. Let us know what you think and help us shape the direction of this feature going forward. }