---
tags: ['post']
post_og_image: 'site'
date: '2009-08-19'  
post_title: Improving performance of concurrent WCF calls in Silverlight applications
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: improving-performance-of-concurrent-wcf-calls-in-silverlight-applications
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---




Many people notice an apparent degradation of latency of concurrent WCF calls in a Silverlight application compared to a similar code in a Windows Console application. This post explains this phenomenon and shows how to optimize performance of concurrent calls in Silverlight.  

WCF proxies in Silverlight applications use the SynchronizationContext of the thread from which the web service call is initiated to schedule the invocation of the async event handler when the response is received. When the web service call is initiated from the UI thread of a Silverlight application, the async event handler code will also execute on the UI thread. Given that there is only one UI thread in the Silverlight application, and given it has other chores (primarily managing the UI itself), the average latency of multiple concurrent WCF calls *as measured by the code executing on the UI thread* may appear to be long. This is because the execution of the event handlers for individual calls needs to be serialized on a single thread and interlaced with other tasks that thread has to perform. The benefit of this model is that the code in the event handler can directly manipulate controls on the page, which is a privilege reserved for code executing on the UI thread. The example below shows how to measure an average latency of multiple WCF calls initiated from a UI thread:  

```
int responseCount;       
int totalLatency;        
const int MessageCount = 100;     

void MainPage_Loaded(object sender, RoutedEventArgs e)       
{        
    TestService.ServiceClient proxy = new TestService.ServiceClient();        
    proxy.HelloCompleted += new EventHandler<WcfPerf.TestService.HelloCompletedEventArgs>(proxy_HelloCompleted);        
    for (int i = 0; i < MessageCount; i++)        
    {        
        proxy.HelloAsync("foo", Environment.TickCount);        
    }        
}     

void proxy_HelloCompleted(object sender, WcfPerf.TestService.HelloCompletedEventArgs e)       
{        
    int end = Environment.TickCount;        
    this.responseCount++;        
    this.totalLatency += end - (int)e.UserState;              
    if (this.responseCount == MessageCount)        
    {        
        this.Log.Text = ((double)(this.totalLatency) / MessageCount).ToString();        
    }        
}     

```
  

In order to avoid synchronizing the invocations of event handlers for individual calls on a single UI thread, a worker thread can be used to initiate the web service calls. Newly created worker threads do not have a SynchronizationContext set, which means event handler code for every response from the server will execute on its own thread. This greatly reduces contention in the client application and reduces the average latency of a series of concurrent calls. Consider the code below which uses the worker thread approach:  

```
int responseCount;       
int totalLatency;        
const int MessageCount = 100;        
Thread workerThread;     

void MainPage_Loaded(object sender, RoutedEventArgs e)       
{        
    this.workerThread = new Thread(this.StartSending);        
    this.workerThread.Start();        
}     

void StartSending()       
{        
    TestService.ServiceClient proxy = new TestService.ServiceClient();        
    proxy.HelloCompleted += new EventHandler<WcfPerf.TestService.HelloCompletedEventArgs>(proxy_HelloCompleted);        
    for (int i = 0; i < MessageCount; i++)        
    {        
        proxy.HelloAsync("foo", Environment.TickCount);        
    }        
}     

void proxy_HelloCompleted(object sender, WcfPerf.TestService.HelloCompletedEventArgs e)       
{        
    int end = Environment.TickCount;        
    lock (this.workerThread)        
    {        
        this.responseCount++;        
        this.totalLatency += end - (int)e.UserState;        
    }        
    if (this.responseCount == MessageCount)        
    {        
        this.Dispatcher.BeginInvoke(delegate        
        {        
            this.Log.Text = ((double)(this.totalLatency) / MessageCount).ToString();        
        });        
    }        
} 

```
  

My ad-hoc measurements indicate the average latency of the worker thread approach is about 20% of the average latency of the UI thread approach. The downside of this approach is the more complex way of manipulating the UI controls on the page. The worker thread has to explicitly schedule the code that manipulates the UI to execute on the UI thread, as shown in the proxy_HelloCompleted implementation above.   }