---
layout: post
title: "Let’s build a redis CLI in python"
date: 2020-01-09 17:10:05
categories: [tutorial, redis, python]
comments: true
---

Hi, today I would like to guide you through an exercise of building a Redis client in python. Underway, we will learn a bit about command line interface app, encoding/decoding a simple network protocol message and network programming using python asyncio.

<!--more-->

![](https://drive.google.com/uc?id=1JF7Co5iRc8xiXnesqZm3sPdjT76ewbEn)

**Credits**: This post is inspired by a rust tutorial streamed by [@ryan_levick](https://twitter.com/ryan_levick){:target="_blank"} a few months ago. You should check him out if you are excited about rust.

## Setup

 To interact with Redis, we will install and run a Redis server. Currently, the easiest way of doing so is to use docker. Hoping you are already using it, got to your command line and run `docker run -p 6379:6379 redis` this will pull and start the Redis docker image that will be accessible on `127.0.0.1:6379`. 
From here you could install the official redis-cli to interact with your local Redis server. However, for this post, we will jump right into building our Redis client.
**Note**: We are not inventing anything new here. The tool we are about to build already exist here as  [Redis-cli](https://redis.io/topics/rediscli){:target="_blank"}. We are just learning by reinventing part of the wheel.
Now let's create a python project with the following folder structure.
```
/py-redis
    │   README.md
    │   main.py
    │   resp.py
    │   tests.py
    └───/venv
```
In this structure, `README.md` will contain all the documentation ceremony. `main.py` will serve as the entry point of our CLI program. `resp.py` will contain anything related to interacting with Redis. As an FYI, the Redis protocol is called `RESP` hence the name `resp.py`. `tests.py` will hold some basic unit tests.

## Let's implement the main CLI program

I like designing software libraries starting from the client code. This helps come up with a good API while getting early feedback on the API for iterative enhancement. As the code is relatively small I will just paste and run you through.

<script src="https://gist.github.com/evanxg852000/206d7c84c049fff1e8446457c48b5771.js"></script>

Since we are using asyncio for all io and network interactions, we will import the python standard library implementation of asyncio. Next, we will import a `Client` class from `resp` module. This is our Redis client library that interacts with Redis. We will go over its implementation shortly but as you can see from how it's been used. it's external API consists of:

 - A constructor: That takes the async event loop, this is important as we will be calling async routines in our `Client` class and we don't want those async routines to be called in another event loop context.
 - An execute method: will execute commands on the Redis server.
 - A close method: will close the Redis server connection if any.
    
Next, let's create an async function `main_client` that will run as the main function in the async event loop. This function takes a prompt parameter that will be displayed on the command line and the actual async event loop. In `main_client`, we create an instance of `Client` passing in the loop; We then start an infinite loop until the command `.exit` is entered by the user. This serves as a way of distinguishing Redis commands from our CLI tool. Commands starting with a dot are known as CLI commands and any other will be treated as Redis command, therefore, redirected to the Redis server. At the end of the loop, we make sure to close any open Redis connection.

The main loop is created from `asyncio.get_event_loop()` and we request asyncio to run our `main_client` function until completion by passing the required parameters. Finally, we close the loop for cleanup purposes.

## Building the redis client

Let's now implement the `Client` class code in this section.

<script src="https://gist.github.com/evanxg852000/98932bfdad32fce55558f8de83477e80.js"></script>

In this class, we assign the loop to a class instance variable, that loop reference will be used for calling async routines that require the executor loop. We also create and assign `None` to `reader` and `writer`  instance variables. These variables represent the TCP connection buffers that we will use to communicate with the Redis server.
In the `execute` method, we check reader value to know if there is a connection or not. If no connection exists, we force the user to connect first by rejecting any other command than `.connect ip:port`. 
For an existing connection, we split the command by space to encode the  redis command. This step is really specific to the way RESP protocol works and will be discussed shortly in the encoder/decoder section of this post.
On this line `self.writer.write(encode(cmd).encode())` ,  we encode the command and write it to the TCP io buffer. The first encode call is an encoding routine we will write as our resp implementation. The second converts the string data into bytes since we want to write bytes of data into the io buffer instead of raw strings. This is followed by reading 1024 bytes from the reader io buffer. We convert these bytes into a string and decode it with our resp decoder implementation. We finally return the decoded response from the Redis server. The last method `close` checks if there is an already opened connection and closes it. 
Note: In the context of reading Redis server replies, we are only reading a maximum of 1024 bytes. Meaning any reply that takes more than that will corrupt our communication flow with the server. I will leave this as an exercise for you to fix by reading the full reply from Redis. For now, this will work for replies less than 1024 bytes.

## Working out the encoder and decoder

To communicate with Redis, we need to understand the Redis language or communication protocol called `resp`. Fortunately, resp is a very basic protocol that is easy to implement. First of all, you will need to skim through this [documentation](https://redis.io/topics/protocol){:target="_blank"} for an overview of the protocol and supported data types.
Hoping you have read the resp protocol documentation, you should know that commands are sent to Redis as an array of string. 
The resp data types are:
- Errors: start with `-`
- Simple String: starts with `+`
- Integers: starts with `:`
- Bulk String: starts with `$`
- Arrays: starts with `*`

The following is a simple implementation of resp encoder/decoder in its all glory 😃.  
<script src="https://gist.github.com/evanxg852000/19852b124dfa8b3e95c0883b7f08746f.js"></script>

The `encode` function just checks the python data type of the value passed in and decides to convert to a resp string representation. Notice that we are using a parameter to force in case we need a resp simple string. This is because I could not find a better python type to represent the simple string. Please, let me know in the comments👇.
The `decode` function is as easy as the encoding counterpart. It looks at the first character and constructs the data type to return while extracting the actual value. Note the recursive call for arrays on `items[i-1] = decode(f'{parts[i]}\r\n')` .

## Summary

Now that all features are implemented, you can run this in another terminal window with `python3 main.py`. I assume your Redis server from the setup section is still running. Run some Redis commands and see the magic 😉.

![](https://drive.google.com/uc?id=1TwJj-uw5G5pVZZA2kOTeujJOL4VYYAZu)

The full source code is available on [Github](https://github.com/evanxg852000/blog-codes/tree/master/python/redis-cli){:target="_blank"}. I hope you enjoyed reading this as much as I am currently enjoying writing articles on this blog. Thanks 🙏for reading through & don't forget to give feedback at [@evanxg852000](https://twitter.com/evanxg852000){:target="_blank"} or share it. 

