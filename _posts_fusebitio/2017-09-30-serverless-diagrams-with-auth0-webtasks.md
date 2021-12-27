---
tags: ['post']
post_og_image: 'site'
date: '2017-09-30'  
post_title: Serverless diagrams with Auth0 Webtasks
post_author: Tomasz Janczuk
post_author_avatar: tomek.png
post_image: blog-tomek.png
post_slug: serverless-diagrams-with-auth0-webtasks
post_date_in_url: true
post_excerpt: Tomek on Software - shaken, not stirred
---

A picture is worth a thousand words. And a good sequence diagram can cut the time necessary to explain the behavior of a system tenfold. How often do you need a quick and convenient way to create, modify, and share a sequence diagram with a simple URL?

<img src="tomek-blog/2017-09-30/0.png" class="tj-img-diagram-100" alt="Sequence Diagram Webtask">

With [Auth0 Webtasks](https://webtask.io), you can create such diagrams with a convenient DSL using a web-based editor or a command line tool, and share them with a unique HTTP URL. 

<img src="tomek-blog/2017-09-30/1.png" class="tj-img-diagram-100" alt="Sequence Diagram DSL">

Many people know [Auth0 Webtasks](https://webtask.io) as the quickest way to set up an HTTP endpoint implemented in Node.js. However, the Webtasks technology supports a very powerful concept of [webtask compilers](https://webtask.io/docs/webtask-compilers). Webtask compilers allow you to define a custom programming model or DSL in which users can implement serverless endpoints. In the example above, a custom webtask compiler is used to enable authoring of webtasks in the syntax supported by [js-sequence-diagrams](https://bramp.github.io/js-sequence-diagrams/). 

### Create sequence diagram

We are going to create a sequence diagram using *wt-cli*, the command line tool for managing Auth0 Webtasks. First, install and initialize the tool:

```
bash
sudo npm i -g wt-cli
wt init
```
Then, create a file with the definition of the sequence diagram. It will be used as a source code of the webtask: 

```
bash
cat > diagram.txt <<EOF
Caller->Auth0: First, you get an access token
Note over Auth0: Authenticate and\nauthorize caller
Auth0->Caller: {access_token}
Caller->API: Then, you call the API {access_token, data}
API->Caller: {result}
EOF
```
Lastly, create a new webtask from this file using the [sequence diargam compiler](https://github.com/tjanczuk/wtc#sequence-diagram): 

```
bash
wt create diagram.txt --name diagram \
  --meta wt-compiler=https://raw.githubusercontent.com/tjanczuk/wtc/master/sequence_diagram_compiler.js \
  --meta wt-editor-linter=disabled
```
The *wt-compiler* metadata property instructs Auth0 Webtask runtime to use the code from the specified URL to *transpile* the webtask script (js-sequence-diagram DSL in this case) into a JavaScript function in a form webtask runtime can execute. In this case the compiler will return a function that responds to HTTP GET requests with *text/html* response to show an HTML page that renders the diagram using [js-sequence-diagrams](https://bramp.github.io/js-sequence-diagrams/). You can see the source code of the compiler [here](https://github.com/tjanczuk/wtc/blob/master/sequence_diagram_compiler.js).

The *wt-editor-linter* property disables default JavaScript linting rules, which do not apply to our custom DSL. 

### Sequence diagram webtask

The *wt create* command returns a URL of the webtask endpoint that serves the diagram. You can optionally add the `?theme=hand` URL query parameter to that URL to render the diagram in a hand-written style rather than the default, simple style: 

<img src="tomek-blog/2017-09-30/3.png" class="tj-img-diagram-100" alt="Sequence Diagram Hand Style">

You can launch the Webtask Editor to edit the code of the webtask in the js-sequence-diagram DSL with:

```
bash
wt edit diagram
```
This will open a browser and navigate to the Webtask Editor that allow you to modify the diagram definition: 

<img src="tomek-blog/2017-09-30/1.png" class="tj-img-diagram-100" alt="Sequence Diagram DSL">

### What else can you do with webtask compilers?

Webtask compilers are a powerful concept that enable you to implement serverless endpoints in Auth0 Webtasks using custom programming models and DSLs. You can read more about compilers in the [documentation](https://goextend.io/docs/developer-guide#middleware), and see more samples in [tjanczuk/wtc](https://github.com/tjanczuk/wtc).

### Big picture: Auth0 Extend, and Webtasks

[Webtask.io](https://webtask.io) is a freemium sandbox of the [Auth0 Extend](https://goextend.io) product. Based on the Auth0 Webtasks technology, it offers modern, serverless extensibility solution for SaaS platforms and applications. Think [better webhooks](https://auth0.com/blog/why-is-serverless-extensibility-better-than-webhooks/).
