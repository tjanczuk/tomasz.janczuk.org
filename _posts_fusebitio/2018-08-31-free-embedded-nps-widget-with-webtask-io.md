---
tags: ['post']
post_og_image: 'site'
date: '2018-08-31'  
post_title: Embedded NPS widget with webtask.io
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: free-embedded-nps-widget-with-webtask-io
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---

[Net Promoter Score (NPS)](https://en.wikipedia.org/wiki/Net_Promoter) has been an established tool for measuring customer loyalty since 2003. It is based on asking your customers a single question: how likely are they to recommend your product to others? Research shows that the answer is correlated to the future success of the product. 

<img src="tomek-blog/2018-08-31/0.png" class="tj-img-diagram-100" alt="How likely are you to recommend Extend to your friend or colleauge?">

In this post I will show how you can create a free NPS widget, complete with UI, backend storage, and reporting using [Auth0 Webtasks](https://webtask.io), and embed this NPS widget with one line of HTML into your web application, single page app, or a blog post. 

### Taking the high road

Start by installing and initializing *wt-cli* if you have not already: 

```
bash
npm install -g wt-cli
wt init
```
Next, create your free NPS widget hosted as a serverless endpoint on [webtask.io](https://webtask.io):

```
bash
wt create -n nps-blog \
  -d webtask-compiler \
  --meta wt-compiler=webtask-compiler/nps <<EOF
EOF
```
You will get back a URL we will call *{webtask_url}*. This is the endpoint of your NPS survey. 

Next, use an *iframe* to embed your NPS survey in your web site, single page app, blog post, or wherever you host your HTML:

```
html
<p>How likely are you to recommend MyProduct to your friend or colleague?</p>
<p><iframe src="{webtask_url}" style="border: 0; height: 1em; width: 11em;"></iframe></p>
```
Next, sit back and watch as NPS survey results pour in when your customers visit your site and share their opinion:

<img src="tomek-blog/2018-08-31/1.png" class="tj-img-diagram-100" alt="How likely are you to recommend MyProduct to your friend or colleauge?">

Lastly, to view the results of your NPS survey, navigate to *{webtask_url}/stats*. You will get back a JSON document with the histogram of all answers as well the number of NPS promoters, detractors, passives, as well as the overall NPS score:


```
json
{
  "histogram": {
    "0": 0,
    "1": 0,
    "2": 2,
    "3": 0,
    "4": 0,
    "5": 1,
    "6": 0,
    "7": 2,
    "8": 4,
    "9": 5,
    "10": 6
  },
  "count": 20,
  "avg": 8.05,
  "nps": {
    "promoters": 11,
    "passives": 6,
    "detractors": 3,
    "score": 40
  }
}
```


Congratulations on your NPS score of 40%! MyProduct has a bright future. 

### Test it out

Now that you experienced creating your own, embeeded NPS widget, I need to ask you a crucial question:

> **How likely are you to recommend webtask.io NPS widget to your friend or colleauge?** 
> <iframe src="https://wt-53f70144dc9d7c76455fa91f858d4cec-0.sandbox.auth0-extend.com/nps-blog?size=2em" style="border: 0; height: 4em; width: 22em;"></iframe>
> 

If you answered 9 or 10, you have a moral obligation to [tweet this blog post](https://twitter.com/intent/tweet?text=Embed%20a%20free%20NPS%20survey%20on%20your%20site%20and%20find%20out%20what%20customers%20think%20about%20you!&url=https://tomasz.janczuk.org/2018/08/free-embedded-nps-widget-with-webtask-io.html&via=tjanczuk&hashtags=nps,webtaskio,serverless,product) with a nice comment.

To check the results of the survey above, click [here](https://wt-53f70144dc9d7c76455fa91f858d4cec-0.sandbox.auth0-extend.com/nps-blog/stats).

### Wait. What, how?

Your NPS widget is an HTTP endpoint created with [webtask.io](https://webtask.io). Webtask.io is a sample, freemium deployment of the [Extend](https://goextend.io/webtaskio) platform, a powerful solution for extending and customizing SaaS platforms. 

The NPS webtask responds to HTTP GET requests by serving HTML representing the survey, allowing users to click to enter their answer, and making an HTTP POST request back to itself to register that answer. Survey results are stored in [webtask storage](https://webtask.io/docs/storage), and disambibuated between individual users using an HTTP cookie randomly generated and set if not already provided on the HTTP POST request. This is of course not a waterproof way of ensuring one user can only provide a single answer (since cookies can be purged or anonymous sessions used), but in many situations more than adequate. Cookies also allow the widget to present the user with the last answer they provided on subsequent visits to your site, and change that rating without double-counting. 

The last function of the widget is to respond to HTTP GET requests to *{webtask_url}/stats* with a JSON document that represents the statistical results of the survey, including the NPS score. 

If you are interested in the technical details of the implementation, as well as many other cool things you can do with [Auth0 Webtasks](https://webtask.io), check out [tjanczuk/wtc](https://github.com/tjanczuk/wtc/blob/master/README.md#embeddable-nps-widget).

### Shameless plug

If you enojoy the flexibility of Auth0 Webtasks, you may be interested in the commercial product we've built on top of this technology: [Extend](https://goextend.io/webtaskio). Extend removes friction from the customization and integration of SaaS platforms by providing an embedded scripting experience, ["serverless wehbooks"](https://fusebit.io/blog/2018/03/serverless-webhooks-to-revolutionize-the-saas/). Check it out! 