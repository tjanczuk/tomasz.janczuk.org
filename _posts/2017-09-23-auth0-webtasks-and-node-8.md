---
layout: post
title: Serverless webhooks in Node 8 with Auth0 Webtasks
author: Tomasz Janczuk
tags:
- serverless
- webtask
- webtasks
- auth0
- microservice
- javascript
- nodejs
- webhook
- node8
---

Many people know [Auth0 Webtasks](https://webtask.io) as the quickest way to set up an HTTP endpoint implemented in Node. Until now, you could write your code using Node 4.x with all public NPM modules. 

> This week we enabled support for Node 8.

Here is how you can start writing your webtasks in Node 8.

### Install and initialize the CLI

If you have not done it already, make sure you have *wt-cli* installed and initialized against the current Node 4.x sandbox:

```bash
sudo npm i -g wt-cli
wt init
```

### Create a Node 8 profile

Now create a new *profile* for *wt-cli* that uses the same token and container as the profile created in the previous step, but the sandbox URL of the new Node 8 sandbox: 

```bash
wt init -p node8 \
  --url https://sandbox.auth0-extend.com \
  --token $(wt profile get default --field token) \
  --container $(wt profile get default --field container)
```

### Webtask away

Use the new *node8* profile from the previous step to create webtasks in Node 8: 

```
echo "module.exports = cb => cb(null, process.versions)" > whatnode.js
wt create -p node8 whatnode.js
```

You will get back a URL to your webtask, which you can then call, e.g.:

```bash
tomek$ curl https://tjanczuk.sandbox.auth0-extend.com/whatnode
{"http_parser":"2.7.0","node":"8.5.0","v8":"6.0.287.53","uv":"1.14.1",
"zlib":"1.2.11","ares":"1.10.1-DEV","modules":"57","nghttp2":"1.25.0",
"openssl":"1.0.2l","icu":"59.1","unicode":"9.0","cldr":"31.0.1",
"tz":"2017b"}
```

And of course you can also launch into the Auth0 Extend editor from the command line with:

```bash
wt edit -p node8 whatnode
```

In the editor, you have access to real-time logs, secret management, all NPM modules, and more - all using Node 8.

<img src="/assets/post_images/2017-09-23/0.png" class="tj-img-diagram-100" alt="Auth0 Extend editor">

### Serverless Framework, Auth0 Extend, and Webtasks

[Webtask.io](https://webtask.io) is a freemium sandbox of the [Auth0 Extend](https://auth0.com/extend) product. Based on the Auth0 Webtasks technology, it offers modern, serverless extensibility solution  for SaaS platforms and applications. Think [better webhooks](https://auth0.com/blog/why-is-serverless-extensibility-better-than-webhooks/).

Until recently the freemium Auth0 Extend sandbox at [webtask.io](https://webtask.io) allowed you to use Node 4.x with all public NPM modules to implement serverless webhooks. 

Last week we shipped the [Auth0 Webtasks provider for Serverless Framework](https://auth0.com/blog/serverless-framework-and-auth0-webtasks-hop-on-the-bullet-train/). On this occasion we provisioned a new version of the Auth0 Extend sandbox, based on the latest version of Node 8. 

This post shows how any subscriber to [webtask.io](https://webtask.io) can also use the new, Node 8 environment. Enjoy!
