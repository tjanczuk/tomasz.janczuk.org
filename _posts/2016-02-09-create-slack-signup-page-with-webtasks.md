---
layout: post
title: Create Slack signup page with Auth0 Webtasks
author: Tomasz Janczuk
bottomimage: /assets/images/b_1.jpg
tags:
- webtask
- webtasks
- microservice
- slack
- signup
- invite
- web
- single page
- javascript
---

### So you want to have a public Slack team for an open source project or community?

Someone recently asked me if there is a Slack channel for [Edge.js](https://github.com/tjanczuk/edge) where questions can be asked. I thought it was a great idea to create one as a modern equivalent of IRC (kids, remember IRC?...). I was quick to discover that while creating a new Slack team is easy and free, enabling everyone to participate is not. 

<img src="/assets/post_images/2016-02-09/1.png" class="tj-img-diagram-75" alt="Slack signup page for Edge.js">

Individuals must be invited to a Slack team by e-mail. Out of the box, Slack does not provide any UI experience for requesting an invitation. People must be explicitly invited by the Slack team's admin either via Slack UI or HTTP APIs. Consequently a slew of solutions sprang up to help people create "self-signup" pages, where anyone can enter their e-mail address to have an invitation automatically sent to them. Most of these are either [complex to set up](https://levels.io/slack-typeform-auto-invite-sign-ups/), or require [explicit hosting](https://github.com/rauchg/slackin). 

### Simple Slack signup page with Auth0 Webtasks without backend or hosting

[Auth0 Webtasks](https://webtask.io) allow you to create lightweight UI web apps and microservices without worrying about hosting, yet giving your full flexibility of writing code in Node.js. You can use webtasks to create a Slack signup page for your public Slack team, [like I did for Edge.js](https://webtask.it.auth0.com/api/run/tjanczuk/edgejs-slack-invite): 

<img src="/assets/post_images/2016-02-09/2.png" class="tj-img-diagram-100" alt="Slack signup page for Edge.js">

### Creating your own Slack signup page is easy with webtasks

To create Slack signup page with webtasks, you will need your Slack team's name (e.g. *edgejs*, the part before *.slack.com* in the URL) and the admin Slack token, which you can [obtain here](https://api.slack.com/web#authentication). Now follow these 3 simple steps to create your webtask-driven signup page: 

```bash
npm install -g wt-cli
wt init
wt create https://raw.githubusercontent.com/auth0/webtask-slack-signup/master/slack-invite.js \
    --name {your_slack_team}-signup \
    --capture \
    --secret SLACK_ORG={your_slack_team} \
    --secret SLACK_TOKEN={your_slack_admin_token}
```

Optionally, you can also provide `--secret LOGO_URL={url_to_your_logo}` which will display your custom logo on the Slack signup page. It should be square and not less than 100x100px. 

Use the resulting URL as your Slack signup page and start building a public community around your OSS project!

<img src="/assets/post_images/2016-02-09/3.png" class="tj-img-diagram-100" alt="Slack signup page for Edge.js">

If you are interested in what the webtask code is doing behind the scenes, or customize the UI or signup exprience, check out the [auth0/webtask-slack-signup](https://github.com/auth0/webtask-slack-signup) repo. Since you have the flexibility of writing any Node.js code, your customization options or unlimited. 

### But is that secure?

It is, your Slack admin token is never made public. Read more about [security model of webtasks](https://webtask.io/docs/how).

### What else can I do with webtasks?

Anything you can code up in Node.js really. In practice webtasks are great for implementing microservices, microwebapps (which involve web UI like the signup page above), webhooks, as well as [extending SaaS plarforms with custom code](tomasz.janczuk.org/2015/07/extensibility-through-http-with-webtasks.html). The latter is the how we are using webtasks at [Auth0](https://auth0.com) to enable our customers to customize our platform with Node.js code. 

For another specific example of webtask usage, check out my prior post on [accepting Stripe payments from Single Page Appliacations using Webtasks](https://tomasz.janczuk.org/2016/01/accept-stripe-payments-without-backend-using-webtasks.html). Or, even, better, you can just test it right here by buying me a coffee:

<form action="https://webtask.it.auth0.com/api/run/tjanczuk/coffee4tomek" method="POST" style="margin-bottom: 40px">
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

Enjoy webtasks!