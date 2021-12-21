---
tags: ['post']
post_og_image: 'site'
date: '2016-01-29'  
post_title: Accept Stripe payments without a backend using Webtasks
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: accept-stripe-payments-without-a-backend-using-webtasks
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---

Can I ask you to buy me a coffee? A cup of *tall dark rost* costs $2.14 at my local Starbucks. Just click the button below and use your credit card.

<form action="https://tjanczuk.sandbox.auth0-extend.com/coffee4tomek" method="POST" style="margin-bottom: 40px">
  <script
    src="https://checkout.stripe.com/checkout.js" class="stripe-button"
    data-key="pk_live_CeoSX5oORYP0f5vXtj67o7CI"
    data-image="/assets/images/tj.png"
    data-name="Tomek on Software"
    data-description="A cup of coffee for Tomek"
    data-amount="214"
    data-locale="auto"
    data-panel-label="Buy Tomek coffee"
    data-label="Buy $2.14 coffee for Tomek"
    data-allow-remember-me="false">
  </script>
</form>

Haven't clicked yet? I am not surprised, a payment button on a blog post does not inspire a lot of confidence. I hope you will change your mind after reading this post. 

### Stripe payments on single page apps without backend

The button above is supported by [Stripe](https://stripe.com), a payment processing service. Stripe makes it easy to accept credit card payments on your site without worrying about PCI compliance, the complexity of payment processing, fraud detection, and the general embarassment of leaking credit card numbers of your customers to hackers in North Korea. You never actually have to handle credit card numbers themselves - Stripe does it for you. 

The HTML code I had to place on this page to make the Stripe payment button above is simple and well explained in [Stripe documentation](https://stripe.com/docs/checkout). It goes like this:

```html
<form action="{postback_url_to_my_backend}" method="POST">
  <script
    src="https://checkout.stripe.com/checkout.js"
    class="stripe-button"
    data-key="{my_public_stripe_key}"
    data-image="/assets/images/tj.png"
    data-name="Tomek on Software"
    data-description="A cup of coffee for Tomek"
    data-amount="214"
    data-locale="auto"
    data-panel-label="Buy Tomek coffee"
    data-label="Buy $2.14 coffee for Tomek">
  </script>
</form>
```

A Stripe payment flow initiated by clicking this button looks like this (please excuse my handwriting):

<div class="diagram">
    participant Reader of\nthis blog as Y
    participant Stripe as S
    participant My backend as B
    Y->S: clicks button\n{amount, stripe recipient}
    note over S: collects\ncredit card\ninformation
    S->Y: {token}
    Y->B: automatic form post with {token}
    note over B: pre-configured with\n{secret_stripe_key}
    B->S: execute payment\n{token, secret_stripe_key}
    S->B: confirmation
    B->Y: beautiful thank you note
</div>

The notable advantage of this flow is that I never have to directly handle credit card information on my backend. Credit card data is collected and stored by Stripe and a *token* is issued in exchange, which my backend later uses to refer to the credit card when executing the charge. 

> The notable disadvantage of this flow is that I do need a backend

Backend is necessary to create a trust boundary between the code running in the browser (which anyone can look at), and the code that has access to my precious Stripe secret key. That secret key allows me to access and manage my entire coffee budget using Stripe APIs, and as such I need a backend to protect it from prying eyes. 

What exactly is the piece of code I need to run on the backend to process the form post and execute actual credit card charge so that I can get my coffee? The nice people at Stripe made sure I won't excert myself typing vast amounts of code:

```javascript
var my_private_stripe_key = '...';
var token_from_form_post = '...';
var stripe = require('stripe')(my_private_stripe_key);
   
stripe.customers.create({
    source: token_from_form_post,
    description: 'Coffee for Tomek'
}, function (error, response) {
  // ...
});
```

Now the unfortunate predicament I found myself in is that a blog post like the one you are reading is an example of a single page application with a thin, file-serving backend that does not allow me to run aforementioned piece of Node.js code. 

> You need a separate backend. Or do you?

In order to execute that backend code and get my coffee I would have to find myself a place to host it, make sure it is secure (SSL), monitor it, and babysit it as a service through its useful life. In other words take care of a plentitude of actions commonly associated with running your own backend. I would probably be able to collect my $2.14 faster by singing carols at the corner of Pike Place Market than setting up my own backend to run these few lines of code. And believe me when I say I am a bad singer. 

There's got to be a better way.

### Webtasks to the rescue: run backend microservices without running a backend

The [Webtask](https://webtask.io) platform allows you to run backend code without having to worry about maintaining a first class backend application. When you create a webtask, you specify the *backend code* you want to execute, as well as the *secrets* you want that code to have access to (e.g. a secret Stripe key). The Webtask platform gives you back a URL you can call to execute that backend logic. 

> Webtasks are an ideal mechanism for running lightweight microservices in situations that require a trust boundary between your backend logic and the client code.

Getting started with webtasks is a breeze. First initialize your account:

```bash
npm install -g wt-cli
wt init
```

Next store your webtask code in a file; let's call it *coffee4tomek.js*:

```javascript
var stripe = require('stripe');

module.exports = function (ctx, req, res) {
    stripe(ctx.secrets.stripeSecretKey).charges.create({
        amount: 214,
        currency: 'usd',
        source: ctx.body.stripeToken,
        description: 'Coffee for Tomek'
    }, function (error, charge) {
        var status = error ? 400 : 200;
        var message = error ? error.message : 'Thanks for coffee!'; 
        res.writeHead(status, { 'Content-Type': 'text/html' });
        return res.end('<h1>' + message + '</h1>');
    });
};
```

Now create a webtask by combining the code in *coffee4tomek.js* with the secret Stripe key: 

```bash
wt create coffee4tomek.js --parse-body --secret stripeSecretKey={stripe_secret_key} --parse-body
```

You will get back a webtask URL that may look like this:

```
https://tjanczuk.sandbox.auth0-extend.com/coffee4tomek
```

Take that webtask URL and set is as the form action of the Stripe form from the beginning of this post:

```html
<form  
  method="POST"
  action="https://tjanczuk.sandbox.auth0-extend.com/coffee4tomek">
  <script
    src="https://checkout.stripe.com/checkout.js"
    class="stripe-button"
    data-key="{my_public_stripe_key}"
    and-other-stuff>
  </script>
</form>
```

And you are done! Your payment button is fully functional! Time to rake in a fortune and get caffeinated!

Did I earn my coffee *now*?

<form action="https://tjanczuk.sandbox.auth0-extend.com/coffee4tomek" method="POST" style="margin-bottom: 40px">
  <script
    src="https://checkout.stripe.com/checkout.js" class="stripe-button"
    data-key="pk_live_CeoSX5oORYP0f5vXtj67o7CI"
    data-image="/assets/images/tj.png"
    data-name="Tomek on Software"
    data-description="A cup of coffee for Tomek"
    data-amount="214"
    data-locale="auto"
    data-panel-label="Buy Tomek coffee"
    data-label="Buy $2.14 coffee for Tomek"
    data-allow-remember-me="false">
  </script>
</form>

### Webtasks: backendless microservices

How does it all hang together, and how is it secure? You can read about the details of the [webtask security model](https://webtask.io/docs/how), but here is the gist:

1. The webtask code does not contain any secrets. Secrets are provided to that code at runtime through the *ctx* parameter.  
2. Both secrets and webtask code are specified at webtask creation time through *wt create*, encrypted, signed, and stored for you in an undisclosed location in the Mojave desert. (Behind the scenes the secrets and code are bundled together and cryptographically protected from tampering and disclosure in a structure called *webtask token*).  
3. The URL returned to you is a *webtask URL*. The webtask code behind it that you write behaves like a Node.js HTTP handler and provides you with full control over HTTP protocol (although there are also [simpler webtask programming models](https://webtask.io/docs/model)). This means your webtasks can return HTML, JSON, PNG, bitcoin, or whatever else you desire. 
4. Webtask runtime by default presents your webtask code with the secrets using the *ctx.secrets* property, and with parsed HTTP request body in the *ctx.body* property. In the stripe example above, the *stripeToken* parameter is present because it is included as part of the Stripe form POST. 

### Besides credit card processing through Stripe with no backend, what can you do with webtasks?

Webtasks reduce the friction of running backend logic and make it quick and convenient to create lightweight, HTTP-oriented microservices. 

Similarly to how we have created a trust boundary around our Stripe secret key in order process credit card transactions, you can use webtasks to implemented application specific logic around any other kind of protected resources: provide limited access to one object in AWS S3, allow users to run constrained queries in a MongoDB, create custom ways of processing your Twitter feeds. The possibilities are plenty. 

Another great use of webtasks is to make it very easy and simple to implement webhooks. Since a webtask is invoked with a simple HTTP call and the programming model provides you full control over the HTTP protocol, you have enough flexibility to implement any webhook. For example, you can intercept GitHub push events, react to Stripe payments, or integrate with Slack. 

Last but not least, webtasks as a platform allow you to extend a SaaS product by allowing its users to write custom code. I wrote about [SaaS extensibility through custom code with webtasks](http://tomasz.janczuk.org/2015/07/extensibility-through-http-with-webtasks.html) extensively before. 

So what are you waiting for? Webtask away! And don't forget about my coffee:

<form action="https://tjanczuk.sandbox.auth0-extend.com/coffee4tomek" method="POST" style="margin-bottom: 40px">
  <script
    src="https://checkout.stripe.com/checkout.js" class="stripe-button"
    data-key="pk_live_CeoSX5oORYP0f5vXtj67o7CI"
    data-image="/assets/images/tj.png"
    data-name="Tomek on Software"
    data-description="A cup of coffee for Tomek"
    data-amount="214"
    data-locale="auto"
    data-panel-label="Buy Tomek coffee"
    data-label="Buy $2.14 coffee for Tomek"
    data-allow-remember-me="false">
  </script>
</form>
}