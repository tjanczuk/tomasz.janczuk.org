---
tags: ['post']
post_og_image: 'site'
date: '2018-04-05'  
post_title: Build your own Twitter scheduler
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: build-your-own-twitter-scheduler
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---

Twitter is great for marketing your product or personal brand, but it has one disadvantage: you must actually tweet. 

<img src="tomek-blog/2018-04-05/0.png" class="tj-img-diagram-100" alt="Twitter scheduler">

In the next 2 minutes I will show you how to create a simple yet flexible (and free) Twitter scheduler that allows you to set up your tweets along with a tweeting schedule ahead of time. You can then focus your attention and energy elsewhere, while [Auth0 Webtasks](webtask.io) dilligently follow your tweeting directions. 

### Your own Twitter scheduler

Start by installing and initializing *wt-cli* if you have not already: 

```
bash
npm install -g wt-cli
wt init
```
Next, create your intial tweet schedule in YAML in the *buffer.yaml* file. You can change the schedule later:

```
bash
cat > buffer.yaml <<EOF
tweets:
  - text: "Check out this free Twitter scheduler that uses @auth0_extend and @webtaskio: https://tomasz.janczuk.org/2018/04/build-your-own-twitter-scheduler.html\n\n#nodejs #serverless"
    media: https://tomasz.janczuk.org/assets/images/b_1.jpg
    schedule: 
      - 4/4/2018 09:00 PDT
EOF
```
(More on the format later).

Last step: create a CRON job on [Auth0 Webtasks](https://webtask.io) that will run every 15 minutes, inspect your schedule, and send out any due tweets:

```
bash
wt cron create buffer.yaml -n buffer \
  --schedule 15m \
  --no-auth \
  -d webtask-compiler \
  --meta wt-compiler=webtask-compiler/twitter \
  -s TWITTER_CONSUMER_KEY={YOUR_TWITTER_CONSUMER_KEY} \
  -s TWITTER_CONSUMER_SECRET={YOUR_TWITTER_CONSUMER_SECRET} \
  -s TWITTER_ACCESS_TOKEN_KEY={YOUR_TWITTER_ACCESS_TOKEN_KEY} \
  -s TWITTER_ACCESS_TOKEN_SECRET={YOUR_TWITTER_ACCESS_TOKEN_SECRET}
```
(You must substitute your own Twitter credentials which you can get from [here](https://apps.twitter.com/)). 

That's it. Sit down and watch your tweets beeing sent out. 

### Wait. What, how?

Your Twitter scheduler is an [Auth0 Webtask CRON job](https://webtask.io/docs/cron). It runs every 15 minutes (or whetever frequency you choose). It inspects the tweeting schedule you set via the YAML, and sends out any tweets that became overdue since the last time the CRON job ran. The CRON job stores the time it executed last as well as the status of the most recent tweets that were sent in [webtask storage](https://webtask.io/docs/webtask_storage). All this logic is implemented in a [webtask compiler](https://webtask.io/docs/webtask-compilers) which is defined in the [webtask-compiler NPM module](https://github.com/tjanczuk/wtc#twitter-scheduler). In addition to implementing the actual tweeting logic, the compiler allows the use of a custom DSL (the YAML with tweeting schedule in this case) to be used as the *script* of the webtask.  

For a gentle introduction of the powerful *webtask compiler* concept, check out [Raymond Camden's post](https://auth0.com/blog/expanding-auth0-extend-with-compilers/).  

### Updating the Tweet schedule

Given that the YAML is the *source code* of your webtask, you can simply use the Webtask Editor to modify it. Run

```
bash
wt edit buffer
```
which will bring up the webtask editor allowing you to change the YAML. 

 <img src="tomek-blog/2018-04-05/1.png" class="tj-img-diagram-100" alt="Webtask Editor">

 Make any changes, save, and you are done!

 This is what you can do in the Yaml: 

 
```
yaml
 tweets:
  - text: "I just installed a free Twitter scheduler that uses @auth0_extend and @webtaskio.\n\nCheck out https://github.com/tjanczuk/wtc#twitter-scheduler\n\n#nodejs #serverless"
    media: 
      - https://tomasz.janczuk.org/assets/images/b_1.jpg
      - https://tomasz.janczuk.org/assets/images/b_2.jpg
    schedule: 
      - 4/8/2018 09:00 PDT
      - 4/8/2018 15:00 PDT
  - text: "You can do amazing things with webtask.io!\n\n#nodejs #serverless"
    media: https://tomasz.janczuk.org/assets/images/b_2.jpg
    schedule: 4/5/2018 12:00 PDT
```

 1. The top level object must contain the *tweets* array with elements representing individual tweets.  
 2. Each tweet must contain the *text* of the tweet, and a *schedule* at minimum. It may also contain the *media* to attach to the tweet. 
 3. The *schedule* is a single date or an array of dates that tweet will be sent on. You can use any format here that is accepted by Node's `new Date(...)` constructor, like the simple one shown above. It is a good idea to specify the time zone.  
 4. The *media* element, if present, is a single URL or an array of up to 4 URLs. These must be publicly accessible URLs that serve the media (typically an image) to attach to your tweet, with correct *Content-Type* header.  

### Get the status

 You can easily check the status of your Twitter scheduler by navigating to your webtask's URL in the browser: 

<img src="tomek-blog/2018-04-05/2.png" class="tj-img-diagram-100" alt="Status">

The *schedule* element shows the JSON representation of your YAML tweeting schedule, or an error if the YAML could not be parsed. The *plan* element specifies the Tweets that would be sent if the CRON job were to run *now*. 

The last part of the status specifies the 20 most recently sent tweets: 

<img src="tomek-blog/2018-04-05/3.png" class="tj-img-diagram-100" alt="Tweet History">

For each tweet, the *result* element tells if the tweet was sent successfuly or not, and the unique tweet id in case you want to automate further processing. 

### Force the execution, now

Instead of waiting for the CRON job to execute, you can force execution of your schedule. This is done by simply navigating to the webtask URL with a *?run* query string appended, e.g.: 

```
https://{your_container}.sandbox.auth0-extend.com/{your_webtask}?run
```
As a result, all the overdue tweets since the last execution (either by the CRON job or manually) will be sent. 

### Shameless plug

If you enojoy the flexibility of Auth0 Webtasks, you may be interested in the commercial product we've built on top of this technology: [Extend](https://goextend.io?utm_source=blog&utm_medium=post&utm_campaign=blog-tomek&utm_content=2018-04-05-twitter-scheduler). Extend removes friction from the customization and integration of SaaS platforms by providing an embedded scripting experience, ["serverless wehbooks"](https://fusebit.io/blog/2018/03/serverless-webhooks-to-revolutionize-the-saas/). Check it out!