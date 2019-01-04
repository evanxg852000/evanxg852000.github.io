---
layout: post
title: "Let’s build a NodeJS micro framework (Part I)"
date: 2019-01-04 10:10:00
categories: [tutorial]
comments: true
---

*Learning to build an express like NodeJS framework …*

<!--more-->

![](https://cdn-images-1.medium.com/max/1600/1*0V3KOPgr9EzfaHoV9vT3rg.jpeg)

Frameworks and libraries are great pieces of software we rely on to build
awesome products. As software developers, knowing how they work from the ground
up can help us develop better software. In this three-part tutorial series, I
will guide you through the implementation of an express like web framework,
while we won’t implement all the ins and outs of the
[express](https://expressjs.com) framework, we will implement two of the most
important components of a micro framework.<br> By the end of this tutorial, you
will have a working framework providing the following high-level features: URL
routing, middleware support, template engine with inheritance. I have creatively
named the framework **micro**, sweet name, right?

We all agree that practicing is the best way to master whatever skill we want to
acquire. I highly encourage you to fire up Vim, VSCode or whatever you are
comfortable with writing code.

**Notes**:

* I will be using some ES6 features available in NodeJS (≥**v8.*.***).
* Please don’t use this as an opportunity to create yet another framework; we
already have too many of them.
* If you feel like jumping ahead of the article, the full source code is
accessible on [Github](https://github.com/evanxg852000/njs-micro)

Ready, set, let’s go!!!

To build a developer friendly framework, we first need to think about the API.
How will software developers use it? With the plethora of frameworks at our disposal to get inspiration from, this should be the easiest thing to do. Our implementation
will closely mimic the express framework API.

<script src="https://gist.github.com/evanxg852000/fc20a7bbe4a5fab4ab8c5ec37edbfe76.js"></script>

First of all, import the framework and instantiate it as `app` passing config
parameters:

* The `env`parameter is the environment of the application. It will be used by the
framework to apply some optimizations and other magics.
* The`templates` parameter specifies the folder where template files are located.
* The `port`parameter: I am sure you've guessed that one already.

<script src="https://gist.github.com/evanxg852000/7c9b5ee2f6d824b29a36a0cce506666e.js"></script>

We access the `router`member of the framework instance to set up a route
handler. The `all` method will set up a handler that handles most HTTP verbs
(e.i **GET**, **POST**, **PUT**, **DELETE**). This can be useful for an API landing
page. Such a page can help test availability status. It can also be used to
display miscellaneous info such as version, vendor, etc…

To get access to the full router API, the application `route` method is the way
to go. It gives access to the full router instance of the app. Also, note the
use of the `render` method on the HTTP response. This is the only API for using
the framework template engine. I won’t go into the template engine syntax for
now. The rest of the snippet shows all you can do with the framework router
component.

<script src="https://gist.github.com/evanxg852000/bfd58fcae64c5493dc25069eb5729260.js"></script>

Now that we all know the Micro framework API, we can create our project and
start coding. I will assume you can `npm init`, giving any name to your project.
I have decided to name mine `njs-micro`. Make sure your project has the
following structure.

```
    /njs-micro
    │   README.md  
    │
    └───/src
    │   │   index.js
    │   │   micro.js
    │   │   router.js
    │   │   templater.js
    │   │
    └───/test
    │   │   ...
    │   
    └───/templates
        │   index.html
        │   main.html
```

* `njs-micro/src`: all the core source files are located here.
* `njs-micro/src/index.js`: try-it-out file, where we use our framework API (this
is not unit test).
* `njs-micro/src/micro.js`: the core framework file.
* `njs-micro/src/router.js`: the router component of the framework.
* `njs-micro/src/templater.js`: the template engine component of the framework.
* `njs-micro/templates`: the templates file will reside here while developing,
remember that this path is configurable for users of the framework.
* `njs-micro/test.js`: The unit test folder, please refer to
[Github](https://github.com/evanxg852000/njs-micro) as we won’t cover unit test
in this series.

Now open `njs-micro/src/micro.js,` and let’s code the core class of our
micro framework.

<script src="https://gist.github.com/evanxg852000/2f5b429aeb84cd4e89aa8439bad13516.js"></script>

The imports at the top already show that we will need the NodeJS HTTP module as
well as the router and template modules. The `Micro` class has a few methods
that we will describe and implement one at a time. The methods meant for
internal use start with an underscore. Let’s implement the constructor.

<script src="https://gist.github.com/evanxg852000/73c7b19f1b326acbc9862564db33d850.js"></script>

As you can see, we first create defaults config for the instance and combine it
with the configuration provided by the instantiator (developer). It’s important
to know that configuration from the instantiator take precedence. <br> The next
thing we do is create a bare metal NodeJS HTTP server as a private member of the
class: `_httpServer`. By passing the `_handlerRequest` method as the HTTP
request handler, every time the application receives a request, this method will
be invoked. The last two things the constructor takes care of are: Instantiating
a router and a template engine. The template engine needs to know the templates
folder as well as the application runtime environment for basic optimization
purpose.

Next is the `boot` method that will start the internal HTTP server. Some
frameworks might call this method `run`. Here we just call the `listen` method
of the NodeJS HTTP server. The callback parameter is checked to make sure it’s
callable, then called with any potential error raised by the HTTP server.

<script src="https://gist.github.com/evanxg852000/1f5179f0fe003412c853d01c0fdd3797.js"></script>

The last public interface of the `Micro` class is the `route` method. This
method takes a callback parameter, checks if it is callable, then calls it by
passing the router instance. This is how we give access to the router attached
to the current framework instance. Developers can access the full router API
bound to the framework instance.

<script src="https://gist.github.com/evanxg852000/b2ffeb936b6d2745148b94fab85514a9.js"></script>

You may be asking yourself 🤔; How are we giving access to the template engine?
That’s exactly what we are going to handle next.

In the snippet below, The NodeJS HTTP handler is implemented by the
`handleRequest` method. It accepts request and response as parameters. We first
call the `_patchResponse` method. Next, we manually dispatch (forward) the
request to the router holding all the routes handlers. The router will then
decide which handler to call based on what information it finds in the request.
<br> Calling `this._pathResponse(reponse)` before dispatching is really
important. This will dynamically create the template rendering function, attach
it to the response object before handing it over to the dispatch method. This
dynamic way of enhancing or modifying an object provided by another system at
runtime is called [Monkey Patching](https://en.wikipedia.org/wiki/Monkey_patch).
This is very easy to achieve in dynamic programming languages.

<script src="https://gist.github.com/evanxg852000/e67b07090385525ae247a4206faa8c02.js"></script>

I don’t know about you but, I believe we have covered a lot in this part of the
series 💪🏾💪🏾. As it stands, there is no way to check what we have done. This
is one of the reasons TDD was invented; to get early feedback. 

**Assignment**: Write unit-test for this first component while stubbing out or
mocking other components to verify our work so far. You are restricted to use
only these dependencies [`package.json`](https://github.com/evanxg852000/njs-micro/blob/master/package.json#L16-L19).

I hope this will get you out of your comfort zone and set you on the way to be a
rock star developer! Next time we will explore and implement the router
component of our framework. Until then, Happy coding! 👋🏽
