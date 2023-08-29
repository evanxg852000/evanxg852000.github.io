---
layout: post
title: "Two weeks into Zig ⚡"
date: 2023-08-29 15:00:00
categories: [programming]
comments: true
---

It has already been two weeks since I started trying Zig and I believe this is the moment to note things I like and don’t like about the language. As a disclaimer, this is not about which language is better than the other. I consider myself a polyglot software engineer and programming languages are mere tools helping us.

<!--more-->

First, I have worked with many programming languages throughout the years including PHP, C#.NET, Java, Javascript, Golang. Nowadays, I work primarily with Rust, Golang, and Python. None of those tools is perfect and Zig is not going to be perfect either. That’s something I am well aware. Well, Even Ryan Dahl ended up unhappy with what he created a few years back (NodeJS).  

I may not be a programming language design expert but I have seen a lot of patterns from different languages that I know what I want from a programming language.

For the past two weeks, I have been trying Zig by working through two small projects:

- [zitcask](https://github.com/evanxg852000/zitcask): a basic implementation of the [bitcask paper](https://riak.com/assets/bitcask-intro.pdf).
- [monkey-zig](https://github.com/evanxg852000/monkey-zig): An implementation of the [writing an interpreter in go](https://interpreterbook.com/) book.

As you will notice on the Github repositories, things are not clean and tested as they should be. Given I am still learning Zig, I tend to try different patterns for the same thing just to feel the task and learn. These repositories are likely to be updated as I learn more about how a Ziguanas should behave 😂.

Things I like:

- Tagged unions are cool. Not the same as Rust pattern matching but it has kept the nostalgia at bay.
- The lack of garbage collection is great, but having to manually clean up made me avoid heap allocation at all cost throughout. Something I don’t practice when writing Rust.
- The memory leak detector in tests is awesome, tracing back to where the allocation was made is definitely the icing on the cake.
- Introduction of types (`Vector`) for SIMD instructions.

Things I don’t like:

- There are many control flow constructs (loop, if, break) in the language, each only appropriate in certain contexts with some extra syntax.
- Compile time error messages were not helpful in understanding and fixing issues.
- I would like an alias for strings  `[] cont u8` `[] u8` types.

Overall, I have found Zig to be enjoyable to work with especially when you are coming from C programming. It offers a lot of cool things C is missing. I would not be surprised to see a wave of rewrites from C to Zig when it reaches 1.x. 

In the meantime, I  plan to use Zig to deepen certain areas of my system programming skills. Being a new language without many learning resources and reusable libraries, the learning experience is going to be more of “The Hard way” style. This is a good thing for my learning plans. The following are the Zig repositories I plan to study:

- [Rheia](https://github.com/lithdew/rheia): A Bitcoin implementation full of cool low-level stuff.
- [TigerBeetle](https://github.com/tigerbeetle/tigerbeetle): A distributed financial accounting database.

Thanks for reading 👋🏾

