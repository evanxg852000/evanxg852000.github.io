---
layout: post
title: "How I built a serverles runtime for learning"
date: 2020-01-09 17:10:05
categories: [tutorial, golang]
comments: false
---

Send developers on vacation, and they usually return with a product idea; Because that's mostly when their creativity ticks. In my case i came back with a small toy project that helped me understand and materialize what are cloud functions (aka serveless technology) at the core.

<!--more-->

![](https://drive.google.com/uc?id=1TeK8lNa7Z4Xox6PALeOsy8ibtgk8xo_0)

**Disclaimer**: I am no expert in serverless technology, the only drive has been my curiosity of breaking into piece to  understand a technology I have been using for the past 8 months to deploy service. Mostly using tools (Firebase[linl], serverless Framework[link]) that realy abstract the inner working away from me. Also bare in mind this is just toy project for learning therefore very minimal and careless about performence.


## The story 
So I took a week off few days ago from work to relax and change settings. Since we are still in theses hard times COVID-19, I could not do much staying home. So third day into my timeoff, I decided to implement a fun project that I could learn from. I usually head over to Build x in Y[link]. But this time the idea was just there: I wanted to know if I could make my own serverless runtime. Although I love Rust for doing most of my learning these days, choosing golang was this time obvious for the following reasons:
 - Easy to prototype with
 - Many batle tested libraries that will turn out useful
 - Concurrency is a breez
 - Defacto language for Cloud Native apps 
  
## What I learned

In the most basic form, a cloud function is a function or routine written in a programming language that our runtime is going to package in a docker container. The container will serve as a sanboxing mechanism to isolate the third party code from affecting our server environment. Yes, you heard it *server environement*. serverless is not actually serveless 😉.

I implemented my serverless runtime around the Docker container thechnology. The runtime is a golang web application with a single endpoint to create or update the functions. The serveless runtime being in charge of packaging and running code from all sort of companies and developers. Code structures and API protocols should be in place to make those third party code play by the runtime rules. From my learning experience, I think it's more complicated to comme up with these rules than implementing the servless runtime itself. Mostly because as a serverless runtime provider, you want to provide as many programming environement as possible with all sort of differences. 

In my case, I wanted to provide a way of creating cloud functions in Golang and NodeJS but here we will focus on NodeJS.

## Cloud functions structure
As I said previously, we need to dictate the structure and the conventions in order to make things work nicely while being also flexible enogh to support other programming environement in the future. So without beating around the bush much longer, here how a cloud function project should be structured.

- [NodeJS Serverless Project](https://github.com/evanxg852000/node-eserveless-example)
- [Golang Serverless Project](https://github.com/evanxg852000/go-eserveless-example)

```
/node-eserveless-example
    │   .eserveless.yaml
    │   functions.js
    │   package.json
```

The `.eserveless.yaml` is our project manifest that should contain a declaration of functions available in our project and their properties.
The `functions.js` is the entry point when all functions declared in the manifest file should be exported from. the foloring is a listing of the manifest and the function code.

```yaml
#.eserveless.yaml
repo: https://github.com/evanxg852000/node-eserveless-example
runtime: node
functions:
- name: API
  type: http
- name: Ticker
  type: cron
  schedule: "*/1 * * * *"
  meta:
    path: "/usr/bin"
    name: prod 
```

```js
//functions.js
function API(req, res) { //http function
    const ip = res.socket.remoteAddress;
    console.log(`Client ip address is ${ip}.`);
    res.writeHead(200);
    res.end('Hello, World!');
}

function Ticker(envs) { //cron function
    console.log(envs); 
    console.log(`Ticker ticked at ${Date.now()}`);
}

module.exports = {
    API,
    Ticker
}
```

As you can see from the `.eserveless.yaml` file: A project has  repository url, runtime specified as `node` or `go` and an arrays of functions.
A function has a name, a type that can be one of the values `http`, `cron` respectively for http handlers and periodic function scheduled using crontab syntaxe. Additionally metadata can be added to functions. They are available at runtime.
In our `functions.js` code, we export both functions we previously declared in the manifest.

Our toy runtime implemention works on public project available on Github since I was not in mood to start implementing authentication and other git repos provider. So to create or update a project on our serverless platform, you would run the request with `curl` or even better use our cli tool `$ ./eserveless evanxg852000/node-eserveless-example`

```bash
curl -X POST -H "Content-Type: application/json" \
    -d '{ "repository": "https://github.com/evanxg852000/node-eserveless-example" }' \
    http://serverles.webapi.url
```

Once the the runtime service receive this request, the followings happen:
- The git repository is cloned and the hash or the master branch is checked against our database in case there was nothing changed
- The project and functions are created or updated
- The containers associated with the functions are built according to their types [thinking-head]

This last step still need some clarifications thought. How do we consume run code in `functions.js` for instance. The answers is in the other half of the serverless protocol. When packaging the client function code, we provide our main code that will call the function specified in the manifest. The following is and example of such main code for a http funtion type.

```js
//main
const http = require('http');
const functions = require('./functions') 

//create a server object
http.createServer(functions.API)
    .listen(8000);
```
As you can see we import the `functions.js` file here and mount the node http request handler function onto an NodeJS http server.
Pretty simple right, just putting convetions in place. Now usually the main code is a template as we need to replace `API` with the specified function name but you get the idea.

With all this in place, once a function is invoked, we just run the corresponding container, wait for it to be ready if it's a http function, and just redirect the http request to the container. We also need to timeout after sometime to prevent malicious functions from holding the connection longer.


## The missing pieces

As this is a learning project, there are many pieces missing in this implementation:
 - cleaning up container images no longuer needed
 - functions should be able to specify their own timeout (currently this is just hardcode at 30 seconds)
 - Starting & shutting down the container on evry invocation can be costly for functions that are hot a better strategy could be to keep them alive for reuse
 - Certainly many more bugs and conner cases to fix in my toy implementation


## The conclusion

I had fun writting this learning project and exploring some coner cases. It was a great learning experience and I could not resist to share with you. Please give your feedback in the comment and teach me new about this awesome technology. The full source code of our serverless platform is available on [Github](https://github.com/evanxg852000/eserveless-platform){:target="_blank"}. Please feel free to fix some hiden features (aka Bugs).

I hope this post taught you something interesting about serverless and until next time, stay home - stay safe. May God bless & protect us all 🙏.







<script src="https://gist.github.com/evanxg852000/206d7c84c049fff1e8446457c48b5771.js"></script>


