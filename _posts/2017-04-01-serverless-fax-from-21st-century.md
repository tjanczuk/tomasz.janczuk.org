---
layout: post
title: Serverless fax from the XXI century
author: Tomasz Janczuk
tags:
- serverless
- faas
- webtask
- webtasks
- auth0
- twilio
- microservice
- javascript
- nodejs
- fax
---

When Alexander Bain invented the precursor of a fax machine in 1846, he probably did not expect the technology would be ridiculed a mere century and a half later: 

> "Could you fax over a copy?"  
> "No, I can't fax because of where I live."  
> "Where do you live?"  
> "The 21st century."  

It was not until March 31st 2017 (a day before April Fool's Day) when Twilio breathed a new life into the dying art of fax sending by introducing the [Fax API](https://www.twilio.com/blog/2017/03/twilio-programmable-fax.html). Modern humans can now send faxes by issuing a simple HTTP POST request: 

```bash
curl 'https://fax.twilio.com/v1/Faxes' \
    -X POST \
    -d 'To=%2B15558675309'  \
    -d 'From=%2B15017250604'  \
    -d 'MediaUrl=https://some.server.com/my.pdf' \
    -u AC93c019bcdXXXXXXXXXXXXXXXXX:your_auth_token
```

Unfortunately the API fails to close the last 27 years of generation gap between the invention of PDF in 1990 and today: the *MediaUrl* parameter is expected to point to a PDF document. Hosted. On a server. It is so 2000. 

Let's fix it and create that PDF document on the fly, no servers required.

### Serverless PDF with Webtasks

We are going to use [Auth0 Webtasks](https://webtask.io) to create a Node.js function that accepts text and turns it into a PDF document, and then expose this function as an HTTP endpoint that can be used as the *MediaUrl* parameter in a call to Twilio. No servers, no hosting. Just code. 

First, the function: 

```javascript
'use strict';

const Pdf = require('pdfkit');

module.exports = function(ctx, req, res) {
  const doc = new Pdf();
  
  doc.fill('#232323').fontSize(30)
    .text(ctx.query.title || 'Who needs servers anymore?', { width: 410 });
    
  doc.fill('#232323').fontSize(12)
    .text(ctx.query.body || "When shipping a Webtask only takes minutes.", { width: 410 });
    
  res.writeHead(200, { 'Content-Type': 'application/pdf' });
  doc.pipe(res);
  doc.end();
};
```

There are two ways to turn this function into an HTTP endpoint with [Auth0 Webtasks](https://webtask.io), both simple. You can use the [wt-cli](https://webtask.io/cli) command line tool, or the [Webtask Editor](https://webtask.io/make). We are going to do the latter. Just go to https://webtask.io/make, log in, paste the code of the function into the Webtask Editor window, and save: 

<img src="/assets/post_images/2017-04-03/0.png" class="tj-img-diagram-100" alt="Create HTTP endpoints with Auth0 Webtask Editor">

Voila! You have just created an HTTP endpoint (URL at the bottom of the webtask editor window) that accepts an HTTP GET request with a *title* and *body* query parameters and responds with a PDF document. You can test it in the browser: 

<img src="/assets/post_images/2017-04-03/1.png" class="tj-img-diagram-100" alt="Back to the XXI century">

You can then use this URL as the *MediaUrl* parameter in a call to Twilio, e.g.:

```bash
curl 'https://fax.twilio.com/v1/Faxes' \
    -X POST \
    -d 'MediaUrl=https://skynet.run.webtask.io/pdfme?title=Hello&body=World' \
    ...
```

And with that you are back in the future, in the warm embrace of the XXI century, HTTP endpoints, and serverless computing. But hang on, it gets better. 

### Faxing me softly

Using the Twilio APIs directly to send a fax requires you to specify the API keys as part of the call: 

```bash
curl 'https://fax.twilio.com/v1/Faxes' \
    -X POST \
    -u AC93c019bcdXXXXXXXXXXXXXXXXX:your_auth_token \
    ...
```

This is great if only you will ever send a fax, but what if you want to allow everyone in your office to communicate with lawyers or the federal government over fax using your Twilio account? Fear not, Auth0 Webtasks have you covered. 

We are going to create another Auth0 Webtask. This time it will accept an HTTP POST request with *title* and *body* parameters, delegate the creation of the PDF to the previously created webtask, and then *call the Twilio* API on your behalf. Why is this better? Becuse 

> Webtasks allow you to securely associate secrets with code. 

You can store your Twilio key alongside the created webtask to use it within the code, which enables you to freely share the URL to the webtask itself with folks in your office without disclosing your secret keys.

Here is the webtask code: 

```javascript
'use strict';

const Superagent = require('superagent');

module.exports = function(ctx, cb) {
  Superagent
    .post('https://fax.twilio.com/v1/Faxes')
    .type('form')
    .send({
      MediaUrl: `https://skynet.run.webtask.io/pdfme?title=${ctx.query.title}&body=${ctx.query.body}`,
      To: '+15558675309',
      From: '+15017250604'
    })
    .auth(ctx.secrets.TWILLIO_USER, ctx.secrets.TWILLIO_TOKEN)
    .end((e,r) => cb(e));
};
```

You will notice that *MediaUrl* is pointing to the URL of of the previosuly created webtask that turns title and body parameters into a PDF document. But you will also notice that the basic authentication credentials are sourced from mysterious *ctx.secrets.TWILLIO_USER* and *ctx.secrets.TWILLIO_TOKEN* properties. You can specify them to be securely stored alongside your webtask using the webtask editor: 

<img src="/assets/post_images/2017-04-03/2.png" class="tj-img-diagram-100" alt="Storing secrets with Auth0 Webtasks">

With this webtask created, anyone who has the URL can now send a fax using your Twilio keys: 

```bash
curl https://skynet.run.webtask.io/faxit?title=Hello\&body=World
```

So, are we there yet? Are we fully in the present? Not yet, hang on, it gets better...

### Slash fax it

Using *curl* to send faxes is cool and hip. But we all know that what you really want is to send it directly from Slack with a simple slash command. 

And yes, Auth0 Webtasks have you covered here as well with the [Slash Webtasks](https://webtask.io/slack). Slash Webtasks allow you to serverlessly extend Slack with Node.js. No servers, no hosting. Just code. 

We are going to use slash webtasks to create a slash command in your Slack team to send a fax directly from Slack:

```
/wt faxit {title}: {body}
```

To do this, install [Slash Webtasks](https://webtask.io/slack) on your Slack team, then issue

```
/wt make faxit
```

Next, type the following code in the webtask editor, and save:

<img src="/assets/post_images/2017-04-03/3.png" class="tj-img-diagram-100" alt="Faxing with Slash Webtask">

In the spirit of reusability, we are simply calling into the previously created, fax-sending webtask, after extracting the title and body parameters from the slash command parameters we receive from Slack. 

And with that, we have finally opened a wormhole connecting the today with the days long past:

<img src="/assets/post_images/2017-04-03/4.png" class="tj-img-diagram-100" alt="Slash fax it">

### Conclusion

In this post we've used [Twilio Fax APIs](https://www.twilio.com/blog/2017/03/twilio-programmable-fax.html) with [Auth0 Webtasks](https://webtask.io) to serverlessly send faxes. Then we exposed this functionality as a Slack slash command using [Slash Webtasks](https://webtask.io/slack). 

No servers were harmed in this experiment. 