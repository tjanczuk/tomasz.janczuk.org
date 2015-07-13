---
layout: post
title: Extensibility through HTTP with webtasks
author: Tomasz Janczuk
tags:
- JavaScript
- client
- server
- web 
- webtasks
- auth0
- backend
- webtasks
- platform
- extensibility
---

Customization is what distinguishes platforms from applications. The most powerful form of customization is extensibility through custom code. In this post I will talk about how modern web platforms can benefit from code-based extensibility exposed through HTTP, and how you can make your application or platform extensible using [webtasks](https://webtask.io). 

<img src="/assets/post_images/2015-07-13/3.png" class="tj-img-diagram-75">

### Why customize your app?

Customization makes an app or platform more successful by increasing its scope of use.

<img src="/assets/post_images/2015-07-13/1.png" class="tj-img-small" alt="github" style="width:4em">  

Consider GitHub. At the core GitHub is a content management platform. But with the use of [GitHub webhooks](https://developer.github.com/webhooks/) that platform can be customized to integrate with external systems, for example continuous integration. It makes the overall platform worth more than the sum of its parts. 

<img src="/assets/post_images/2015-07-13/2.png" class="tj-img-small" alt="auth0">

Another example is [Auth0](https://auth0.com), where the webtask technology originates. At the core Auth0 is an identity manegement platform. But with the use of [Auth0 rules](https://auth0.com/docs/rules) implemented as [webtasks](https://webtask.io), the platform can be extended with custom code, providing a wide range of possibilities to interact with external systems and customize identity management behavior. 

### Ways to customize you app

There are different ways to customize an app, from a set of declarative controls all the way to interacting with external systems. Each approach comes with a different trade off between complexity and flexibility. 

> Execution of custom code is the most powerful form of app extensibility

The most powerful extensibility mechanism a platform or app can offer to developers is to enable them to write custom code which affects processing. Such code can either execute locally or remotely. 

Local extension code is installed as a module or software package on the system being extended. For example, continuous integration servers like Strider or Jenkins allow custom extensions to be installed. Such approach is typically limited to single-tenant systems due to the complexity of sandboxing of untrusted code. 

<img src="/assets/post_images/2015-07-13/4.png" class="tj-img-diagram-75">

In contrast, remote extension code is invoked from the platform or app over a predefined protocol, frequently built on top of HTTP. Webhooks offered by platforms like GitHub, Stripe, or Zendesk are good examples of protocol-based remote extensibility. 

<img src="/assets/post_images/2015-07-13/5.png" class="tj-img-diagram-75">

Protocol based extensibility yields itself well to customizing multi-tenant systems since the protocol boundary provides a convenient concept to build a trust boundary around. 

> The choice between local and protocol based extensibility is often a trade off between flexibility and latency

Enabling custom extension code to be run remotely over a predefined protocol offers the higest level of flexibility to developers. It enables decoupling of technology choices, engineering processes, governance, and trust models of the core platform and the extension code. 

### Webhook is half of the story

Webhooks are the prevalent way in which modern web platforms expose protocol-based extensibility. GitHub, Slack, Firebase, Zendesk, Stripe are a few examples of platforms that expose webhooks. 

Yet webhooks are just half of the story. 

> A webhook is like an interface without an implementation, a car without gas

Webhooks merely offer the capability to issue an HTTP request. It is the developer's responsibility to write code that will process that request, find a place to host it, and then maintain that environment as a service. 

Using today's webhooks imposes a lot of engineering taxes on the core task of extending business logic. This is a problem when all you want to do is write code. 

### Webhooks are one-way

Most webhooks offered by web platforms today represent events that are processed asynchronously. 

<img src="/assets/post_images/2015-07-13/6.png" class="tj-img-diagram-75">

The code behind a webhook does not have an immediete effect on the behavior of the core platform. 

> Request-reponse webhooks offer more flexibility in extending systems than one-way webhooks

A more powerful extensibility mechanism would allow the code behind a webhook to return data which the core platform uses to augment its processing. 

<img src="/assets/post_images/2015-07-13/7.png" class="tj-img-diagram-75">

Such request-response webhook model is used by [Auth0 rules](https://auth0.com/docs/rules). By writing custom code that executes *during* the authentication transaction, Auth0 users can augment the transaction results. 

### HTTP extensibility with webtasks

[Webtasks](https://webtask.io) provide a lightweight solution to extending your app or platform through custom code. They address the two problems outlined above by:

* letting you and your users focus on implementing the business logic rather than hosting and maintenance of services,
* making it easy to support request-response extensibility pattern. 

<img src="/assets/post_images/2015-07-13/3.png" class="tj-img-diagram-75">

Webtasks do this by allowing secure, isolated, and fast execution of custom, untrusted code directly over HTTP, *with no prior provisioning*. The body of the HTTP request supplies the Node.js code to be executed, and the response contains the result generated by the code. Consider this simple example: 

```
curl https://webtask.it.auth0.com/api/run/{webtask_container} \
 -H 'Authorization: Bearer {webtask_token}' \
 --data-binary \
  'module.exports = function (cb) { cb(null, { hello: "world" }); }'
```

The *curl* command above issues an HTTP POST request that contains the Node.js code to be executed. The code runs in a [webtask container](https://webtask.io/docs/101) isolated from webtask containers other requests are executing in. 

> Strong webtask container isolation guarantees allow you to use webtasks to securely extend multi-tenant platforms and apps

Webtasks support several [advanced and flexible programming models](https://webtask.io/docs/model), including one with full control over HTTP. The simple Node.js programming model in the example above exports a function that immediately calls the supplied callback to return data to the caller:

```javascript
module.exports = function (cb) {
    cb(null, { hello: "world" });
}
```
 
Webtask can specify arbitrarily complex Node.js code, as long as it completes within a typical lifetime of an HTTP request. As of this writing you can use [well over 600 Node.js modules](https://webtask.io/docs/modules) within the code, and the list is growing. 

### How can webtasks help you?

[Webtasks](https://webtask.io) can help you introduce the most flexible form of customization into your app or platform: extensibility through custom code. 

<img src="/assets/post_images/2015-07-13/8.png" class="tj-img-diagram-75">

Even if you are already offering webhooks, with webtasks you can now greatly improve the experience for *your users*, by allowing them to author extension code directly within your platform rather than forcing them to spend time finding the hosting solution for it and then maintaining it. 

### How do I get started?

We have developed webtasks and have been using them at [Auth0](https://auth0.com) for a long time. In fact, if are using Auth0 today, you are already using webtasks. 

We believe webtasks provide a superior customization experience for developers using our identity management platform. As such we've made webtasks available at [https://webtask.io](https://webtask.io) for anyone to try. If you are interested in using webtasks to extend your own platform or app, get in touch at [support@auth0.com](mailto:support@auth0.com). 