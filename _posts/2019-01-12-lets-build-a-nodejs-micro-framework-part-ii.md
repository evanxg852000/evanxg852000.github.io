---
layout: post
title: "Let’s build a NodeJS micro framework (Part II)"
date: 2019-01-12 09:43:00
categories: [tutorial]
comments: true
---

*Learning to build an express like NodeJS framework …*

<!--more-->

![](https://cdn-images-1.medium.com/max/1600/1*0V3KOPgr9EzfaHoV9vT3rg.jpeg)

In today’s post, we are going to implement the router class of our framework. We
have already discussed the directory structure of the framework in [Part
I](https://medium.com/@evanxg852000/lets-build-a-nodejs-micro-framework-part-i-7cf941f2aec9)
of this series. The router component will be implemented in `src/router.js`.

Our router will be implemented as a regular expression matcher behind the scene
while providing a higher level of route specification syntax. We have already
seen some examples of adding routes to an application using the framework. An
important aspect of routing we won’t implement in Micro is URL generation from
routes. This should be an exercise for active readers.

So without further ado, let me show you the skeleton of our router class in its
most glorified form 😝.

<script src="https://gist.github.com/evanxg852000/9d49a73f4c157399cf132db00e800da1.js"></script>

The listing above shows the initialization of the `_routes`, `_prefix` member
variables of the router class to an array and empty string respectively. Perhaps
the most important method is the one named `route`. As you can see this method
accepts three arguments:

* The route specification: a simpler syntax for describing regex matching.
* The methods: a list of HTTP methods allowed on this route.
* The handlers: a list of functions that will be called when a request is dispatched to
this route.

Our objective in this method is to build a route object and add it to the
`_routes` array member variable. In order to put in context the explanation,
let’s suppose we call the `route` method as follows:

```
    router.route(‘/:class/students/:id/:session?’, [‘GET’, ‘POST’], [ _ => true, _ => false])
```

Our `_routes` member variable should end up with the following content:

```
    [{ 
        methods: [ 'GET', 'POST' ],
        regex: /^\/([_a-zA-Z0-9\-]+)\/students\/([_a-zA-Z0-9\-]+)(\/([_a-zA-Z0-9\-]+))?$/,
        params: [ 'class', 'id', 'session' ],
        handlers: [ [Function], [Function] ] }
    }]
```

Please notice how the route object contains all the information we need. The
methods, handlers properties are easy to map, because they are just direct
assignment from the actual parameters received from the call to
`router.route(...)`. In order to understand the remaining properties, let’s
implement the `route` and `group` methods of the router class.

<script src="https://gist.github.com/evanxg852000/691a08e41f49215ac7cef45a6daadf09.js"></script>

In the `route` method, we first check if `_prefix` is not empty. We tweak the
`specs` parameter accordingly by prefixing it. Then we call `_cleanHandlers`
helper method to validate the request handlers making sure they are callable.
This helper method also converts single handler into an array of handlers. Next,
we push into the `_routes` member variable the newly created route object. The
extraction of the params and regex properties of the route happens in the call
and spread operation `…this._patternToRegex(specs)`. Before looking into the
implementation of this helper method though, let's wrap up our discussion on the remaining methods above.

The `group` method accepts a prefix and a callback function, It assigns the
prefix value to `_prefix` and invokes the callback passing the instance of the
router. The `_prefix` is reset to an empty string after the callback invocation.
This technique is a way of maintaining the router prefix while the routes are
being created in the callback. Consequently, any route created in the callback
will end up with the current value of `_prefix`.

The `all` method is one of the many router API methods that allow users of the
framework to add routes without listing the HTTP methods manually. The remaining methods you will find in the repository on [Github](https://github.com/evanxg852000/njs-micro) are: `get`, `post`, `put` and `delete`.

Now that all these noisy methods are out of the way, let's focus on the most
important methods of our router class.

* `_patternToRegex(specs)`
* `_dispatch(request, response)`

The `_patternToRegex(specs)`, converts a route specification to a regular
expression. I should have named this `_specToRegex`, I was just not willing to
refactor this time. It’s just a toy project after all 😃. Let’s not waste time
on this matter and show the implementation.

<script src="https://gist.github.com/evanxg852000/c4bb44d90922151c24dfe4fa79aa15d0.js"></script>

In this method, we first initialize the `regex` and `params` to an empty string
and empty array respectively. Next, we break the specification into parts and
build each part as a small chunk of the regular expression that will be
generated. Looping through the parts, we first skip any empty part. This takes
into account trailing forward slashes. Next, we check if this part is a route
parameter (e.i does it starts with a colon). If yes, we construct a simple regular
expression that accepts characters allowed in a URL while taking into account
the optional specifier `?`. Otherwise, we just add the part to the regular
expression. Please note how we add the name of the route parameter by cleaning
it and pushing into the `params` array. Finally, we create a Javascript RegExp
object from our generated `regex` string and return an object made of: the regular
expression object and the route parameter names array.

The last bit we need to tackle in this `Router` class is the `_dispatch` method.
Recalling from [Part
I](https://medium.com/@evanxg852000/lets-build-a-nodejs-micro-framework-part-i-7cf941f2aec9),
this is the method in charge of finding the appropriate handler for
processing an incoming HTTP request. This method might seem a bit tricky. Let’s
first show the implementation and then explain how it works.

<script src="https://gist.github.com/evanxg852000/2edc52d34fdefa2efc520d52841900c0.js"></script>

In this method, we will receive the NodeJS HTTP server `request` and `response`
object as parameters. Recall from [Part
I](https://medium.com/@evanxg852000/lets-build-a-nodejs-micro-framework-part-i-7cf941f2aec9)
that we patched the response with a `render` method for the purpose of
template rendering. In the above listing, we essentially do three things:

First, we try to find a matching route based on the URL property of the request.
If no match is found, we simply reply with a `404` status. Next, If a match is
found, we try to extract all the route params value from the matching URL and
map them to the params name. We use the regular expression captured groups to
ignore those with value `undefined` or value starting with a slash. We then loop
through params names to build a key/value pair between the names and the values
that we finally assign to `request.params`. This makes our second act of
patching an object from the NodeJS core library🕺. Finally, we need to build a
chain of handlers out of the matched route handlers array. Again let’s recall
this snippet taken from [Part
I](https://medium.com/@evanxg852000/lets-build-a-nodejs-micro-framework-part-i-7cf941f2aec9).

<script src="https://gist.github.com/evanxg852000/84d911e94cfcd3e8df2ada0007d17aed.js"></script>

For the `next` handler to be invoked, The developer needs to explicitly call the
next argument received in the route handler. To achieve this, we need to embed
the handlers inside each other and end up with the first handler being the
top-level handler. Basically, we want to go from a flat array of handlers to a
[Russian dolls](https://en.wikipedia.org/wiki/Matryoshka_doll) like data
structure of handlers. We could implement this with a Stack or a Linked List.
However, it feels more natural to explain this using the Russian dolls
approach.


<iframe width="718" height="393" src="https://www.youtube.com/embed/-xMYvVr9fd4" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


Since we need to start building our handler chain from the last handler, we copy
and reverse the handlers from the matched route `const handlerStack =
route.handlers.slice().reverse()`. It’s very important to make a copy, because
this chain is built specifically for the current HTTP request being processed.
This chain will embed data (request, response) specific to the current HTTP
request.

To make things easier, we adopt a bottom-up approach. We first build the last
handler which will be embedded into the first to last handler which in turn
will… and so on. We loop through `handlerStack`: In the first iteration,
`nextHandler` is `undefined`. This makes sense because, inside the last handler,
the `next` parameter should be `undefined` as there is no subsequent handler in
the chain. In the loop, we first declare `lastNext` and assigns it the value of
`nextHandler`. After that, we need to encapsulate the current handler in the
form of `next` #L28. Notice how we are passing all the current request data we
need to `handler(request, response, lastNext, …params)` as well as the
previously constructed next handler stored in `lastNext`. We also remember
to reset `nextHandler` via this assignment `nextHandler = next`afterward.

Once outside of the loop, `nextHandler` is a function pointing to the root of
the handler chain. In other words, this is the biggest Russian doll that
encapsulates the remaining once. The only thing left to do here is to invoke
that top-level handler and mark the request as successfully handled
`handledRequest = true`. 

**Notes**: *In a real world scenario, just imagine how you would arrange the
Russian dolls in your backpack. Packing is the process of building the handler
chain, Unpacking is the process of calling the top level handler.*

This previous section concludes the implementation details of our Router
component. Ultimately, a production-grade router will require more validation,
as well as URL generation from named routes. However, our implementation already
has the core features of any production-grade HTTP router component.

Congratulation for making it through 👏 👏 👏. I reckon that implementing the
handler chain gets a bit difficult because of the concepts comming
into play. Please review the following:

* Data Structure: Linked Lists, Stacks
* Javascript: Closures, Function Composition

Next time, we will explore and implement the template engine component of our
framework. Until then, Happy coding! 👋🏽
