---
layout: post
title: "How I built a serverless runtime for learning"
date: 2020-06-28 10:10:05
categories: [tutorial, golang]
comments: true
---

Send passionate developers on vacation, and they usually return with a product idea. Because that’s mostly when their creativity ticks. In my case, I came back with a small toy project that helped me understand and materialize what are cloud functions (aka serverless technology) at the core.

<!--more-->

![](https://drive.google.com/uc?id=1TeK8lNa7Z4Xox6PALeOsy8ibtgk8xo_0)

**Disclaimer**: I am no expert in serverless technology. The only drive has been my curiosity of breaking into pieces to understand a technology I have been using for the past 8 months to deploy services. Mostly using tools like [Firebase](https://firebase.google.com/docs/functions){:target="_blank"} and [Serverless](https://www.serverless.com/){:target="_blank"} that really abstract the inner working away from me. Also bear in mind that this is just a toy project for learning therefore very minimal and careless about performance.

## Story 
So I took a week off a few days ago from work to relax and change settings. Since we are still in theses hard times of COVID-19, I could not do much staying at home. Third day into my time off, I decided to implement a fun project that I could learn from. I usually head over to [Build your own X](https://github.com/danistefanovic/build-your-own-x){:target="_blank"}. But this time the idea was already clear in my head. I wanted to know if I could make my own serverless runtime. Although I love Rust for doing most of my learning these days, choosing Golang was this time obvious for the following reasons:
 - Easy to prototype with
 - Many battle-tested libraries that will turn out useful in the long run
 - Concurrency is a breeze (Well Golang is easy by nature)
 - Defacto language for Cloud-Native apps 
  
## Lessons learned

In the most basic form, a cloud function is a function or routine written in a programming language that our runtime is going to package in a docker container. The container will serve as a sandboxing mechanism to isolate the third-party code from affecting our server environment. Yes, you heard it **server environment**. Serverless is not actually serverless 😉.

I implemented my serverless runtime around the Docker container technology. The runtime is a Golang web application with a single endpoint to create or update the functions. A serverless runtime being in charge of packaging and running code from all sorts of companies and developers. Code structures and API should be in place to make those third-party code play by the runtime rules. From my learning experience, I think it's more complicated to come up with these rules and API than implementing the serverless runtime itself. Mostly because as a serverless runtime provider, you want to provide as many programming environments as possible with all sorts of differences.
In my case, I wanted to provide a way of creating cloud functions in Golang and NodeJS. However, this post will focus on the NodeJS bit and provide a link to the Golang code.

## Cloud functions structure
As I said previously, we need to dictate the structure and the conventions in order to make things work nicely while being also flexible enough to support other programming environments in the future. So without beating around the bush much longer, here is how a cloud function project should be structured for our runtime.

- [Runtime](https://github.com/evanxg852000/eserveless-platform)
- [Sample NodeJS functions](https://github.com/evanxg852000/node-eserveless-example)
- [Sample Golang functions](https://github.com/evanxg852000/go-eserveless-example)

```
/node-eserveless-example
    │   .eserveless.yaml
    │   functions.js
    │   package.json
```

The `.eserveless.yaml` is our project's manifest that should contain a declaration of functions available in our project and their properties.
The `functions.js` is the entry point where all functions declared in the manifest file should be exported from. The following is a listing of the manifest and the function code.

<script src="https://gist.github.com/evanxg852000/c1d38a4118aa9d94f72a83e6ea8b527a.js"></script>

<script src="https://gist.github.com/evanxg852000/856204668a8823007ccc2be89bd83fe9.js"></script>

As you can see from the `.eserveless.yaml` file: 
- A project has a repository URL, a runtime specified as `node` or `go` and an array of functions.
- A function has a name, a type that can be one of the values `http`, `cron` respectively for HTTP handlers, and periodic function scheduled using crontab syntax. Additionally, metadata can be added to functions. They end up being available at runtime to your code as environment variables. In our `functions.js` code, we export both functions we previously declared in the manifest.

Our toy runtime implementation works on public projects available on Github since I was not lazy to start implementing authentication and/or support for other git providers. So to create or update a project on our serverless platform, you would run a `POST` request with `curl` or even better use our CLI tool `$ ./eserveless evanxg852000/node-eserveless-example`.

```bash
curl -X POST -H "Content-Type: application/json" \
    -d '{ "repository": "https://github.com/evanxg852000/node-eserveless-example" }' \
    http://serverles.webapi.url
```

Once the runtime service receives this request, the followings happen:
- The git repository is cloned and the hash of the master branch is checked against our database in case there was nothing changed.
- The project and functions are created or updated.
- The containers associated with the functions are built according to their types 🤔.

This last step still needs some clarifications thought. How do we consume code in `functions.js` for instance? The answer is in the other half of the serverless protocol. When packaging the client function code, we provide the main code that will call the functions specified in the developer or third-party code. The following is an example of such a main code for a HTTP function handler type.

<script src="https://gist.github.com/evanxg852000/5d0376fe4010daa9b543dceb561d3a92.js"></script>

As you can see we import the `functions.js` file here and mount the HTTP request handler function onto a NodeJS HTTP server.
Pretty simple right, just putting conventions in place. Usually, the main code is a template as we need to replace `API` with the specified function name when building; hard coding it won't make it dynamic.
In our runtime implementation, these templates files are located inside the `runtimes` [folder](https://github.com/evanxg852000/eserveless-platform/tree/master/runtimes){:target="_blank"}. 

With all in place, once a function is invoked: we just run the corresponding container, wait for it to be ready if it's a HTTP function before redirecting the HTTP request to the container. We also need to timeout after some time to prevent malicious functions from holding the connection longer. Eventually harming our service.

## Missing pieces

As this is a learning project, there are many pieces missing in this implementation:
 - Cron functions are not automatically reloaded.
 - Cleaning up container images no longer needed.
 - Functions should be able to specify their own timeout (currently this is just hardcode as 30 seconds).
 - Starting & shutting down the container on every invocation can be costly for functions that are hot. A better strategy could be to keep them alive for reuse.
 - Error handling and security features are needed for production-grade runtime.
 - Certainly, many more bugs and conner cases to fix in this toy implementation.

## Short demo

<iframe class="video" width="720" height="425" src="https://www.youtube.com/embed/0kF_P5H4-Ec" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Conclusion

I had fun writing this learning project and exploring some corner cases. It was a great learning experience and I could not resist sharing. Please give your feedback in the comments and share your experience working with this awesome technology. The full source code of our serverless platform is available on [Github](https://github.com/evanxg852000/eserveless-platform){:target="_blank"}. Please feel free to fix some hidden features (aka Bugs).

I hope this post taught you something interesting about serverless and until next time, stay safe. May God bless & protect us all 🙏.
