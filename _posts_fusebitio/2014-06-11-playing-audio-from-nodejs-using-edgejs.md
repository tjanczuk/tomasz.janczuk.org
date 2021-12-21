---
tags: ['post']
post_og_image: 'site'
date: '2014-06-11'  
post_title: Playing audio from Node.js using Edge.js
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: playing-audio-from-node.js-using-edge.js
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




The [Edge.js](http://tjanczuk.github.io/edge) project allows you to use .NET Framework inside of a Node.js application. Why would you *ever* do that? [Scott Hanselman](https://twitter.com/shanselman/status/461532471037677568) puts it this way:  

 ![image](http://lh4.ggpht.com/-aoOIwmnj2Wg/U5jyWA1XAFI/AAAAAAAAD-I/DUnChudhIFE/image_thumb%25255B4%25255D.png?imgmax=800)   

One such problem is playing audio. Node.js core does not support this functionality, so one must resort to writing a native extension in C/C++. You can dust off that Stroustrup book, tool up for memory leak detection, prepare for segfaults\avs, get yourself a bucket of coffee, and [plow on to write some serious C code](https://github.com/TooTallNate/node-speaker/tree/master/deps/mpg123/src).  

Alternatively, you can do it with two lines of C# code…    

### Enter Edge.js  

… and then call into these two lines of C# code from Node.js using Edge.js: 
 

{% highlight javascript linenos %}
   var edge = require('edge');

var play = edge.func(function() {/*
    async (input) => {
        var player = new System.Media.SoundPlayer((string)input);
        player.PlaySync();
        return null;
    }
*/});

console.log('Starting playing');
play('dday.wav');
console.log('Done playing');

{% endhighlight %}



So what happens here? We are using the System.Media.SoundPlayer class from .NET Framework to play a PCM WAV file (lines 5 & 6). We wrap this logic in a C# async lambda expression (line 4). Then we use the edge.func function of Edge.js to create a JavaScript proxy around this async lambda expression (line 3). Lastly, we call that JavaScript proxy function and pass it the file name of the WAV file to play (line 12). 

Edge.js allows you to call .NET functions from Node.js and Node.js functions from .NET. Edge.js takes care of marshalling data between CLR and V8. Edge.js also reconciles threading models of single threaded V8 and multi-threaded CLR, and ensures correct lifetime of objects on V8 and CLR heaps. And all that happens *within* a single process – Edge does not spawn separate CLR processes. Read more [in the Edge.js documentation](https://github.com/tjanczuk/edge).

Coming back to playing audio. If you run the code above you will notice that the *Done playing* message is only printed to the console after the audio has finished playing. This is because the C# code executes on the singleton V8 thread of Node.js, and the Node.js event loop remains blocked. This is of course unacceptable…

### Enter CLR threads

… so let’s fix it. We need to add two more C# lines to play our audio on a CLR thread pool thread and avoid blocking the V8 thread:

{% highlight javascript linenos %}
var edge = require('edge');

var play = edge.func(function() {/*
    async (input) => {
        return await Task.Run<object>(async () => {
            var player = new System.Media.SoundPlayer((string)input);
            player.PlaySync();
            return null;
        });
    }
*/});

console.log('Starting playing');
play('dday.wav', function (err) {
    if (err) throw err;
    console.log('Done playing');
});
console.log('Started playing');

{% endhighlight %}



Notice how we create a new CLR thread pool thread in line 5, and let that thread play our audio. This leaves the V8 thread free to process whatever other events need processing. Also notice that the *play* JavaScript proxy function can still detect when the audio has finished playing by supplying an async callback in line 14. Edge.js will invoke that async callback only after the C# async lambda expression completes, which happens when the audio playing on the CLR thread pool thread has finished playing and the thread terminates in line 8. The fact that the Node.js event loop remains unblocked is evidenced by the *Started playing* message from line 18 showing up before the *Done playing* message from line 16.



At this point we seem to be done. While we wait for the folks cranking C code to finish (ETA: one more week), we can indulge in a more fancy experiment.

### Enter closures

Now that we can play a simple WAV file asynchronously, how about adding some more control over the experience. Let’s have a way to start and stop playing the audio asynchronously at any time. 

This calls for one of the more interesting features of Edge.js: the ability to marshal function proxies between V8 and CLR boundary. Moreover, functions exposed from CLR to Node.js can be implemented as a closure over some other CLR state, which opens interesting possibilities. For example, allowing an instance of System.Media.SoundPlayer to be controlled from Node.js:

{% highlight javascript linenos %}
var edge = require('edge');

var createPlayer = edge.func(function() {/*
    async (input) => {
        var player = new System.Media.SoundPlayer((string)input);
        return new {
            start = (Func<object,Task<object>>)(async (i) => {
                player.Play();
                return null;
            }),
            stop = (Func<object,Task<object>>)(async (i) => {
                player.Stop();
                return null;
            })
        };
    }
*/});

{% endhighlight %}



We are using Edge.js to construct a *createPlayer* JavaScript function (line 3). This function wraps a logic in C# which acts as a factory method. It first creates an instance of System.Media.SoundPlayer (line 5). Then it returns an anonymous object with two functions on it: *play* and *stop.* Both functions are implemented as closures over the instance of SoundPlayer created in line 5, starting and stopping the playback, respectively. 

This is how you can use the *createPlayer* function:

{% highlight javascript linenos %}
console.log('Creating player');
var player = createPlayer('dday.wav', true);

player.start(null, function (err) {
    if (err) throw err;
    console.log('Started playing');
});

setTimeout(function () {
    player.stop(null, function(err) {
        if (err) throw err;
        console.log('Stopped playing');
    });
}, 5000);

{% endhighlight %}





First we create a player in line 2. The *player* is a JavaScript object with two properties: *play* and *stop*. Both are JavaScript functions acting as proxies to the corresponding C# async lambda expressions created within *createPlayer*. You can invoke the *play* function to start playing the audio asynchronously on a CLR thread pool thread. Five seconds later, we can stop the playback by calling the *stop* function (line 10). 

### So what does it all mean?

It means that in many cases it is much easier to write a few lines of C# and use Edge.js rather than a truckload of C code to add “native” functionality to Node.js. 

### Dude, Edge.js surely only works on Windows, why are you wasting my time?

Dear Dude, I am pleased to inform you that Edge.js works on [Mac and Linux as well as Windows](https://github.com/tjanczuk/edge#before-you-dive-in). Yours truly.   }