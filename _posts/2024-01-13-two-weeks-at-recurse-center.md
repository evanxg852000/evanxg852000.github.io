---
layout: post
title: "Two weeks at Recurse Center"
date: 2024-01-13 07:00:00
categories: [recurse, programming]
comments: true
---

If you've been following me on Twitter, you may know from a previous tweet that I was accepted to the [Recurse Center](https://www.recurse.com/) in the Winter 2 '24 batch, which started in the middle of the week on January 3rd.

<!--more-->

![](https://d29xw0ra2h4o4u.cloudfront.net/assets/public_recurser_working-973845079b2a8523c4c802b8f4edba8f4e76fa4769aaffe5a8afe4a8648ca397.jpg)

<small>*Image source: https://www.recurse.com/about*</small>

It was enjoyable to see all the passionate recursers around and hear about the exciting projects they are planning to work on. I had crafted my plan a while ago and truly enjoyed the official kick-off. However, truth be told, I had already started my batch on December 21st during our pre-batch welcome call. Excited to hit the ground running, I took that opportunity to set up things and start coding on my projects.

I have planned to work on so many things that I don't think I will fit them all into a single batch, but that's fine given that at the Recurse Center, you never truly graduate. The most important thing is to avoid spreading hands into too many projects at the same time. However, for the past two weeks, I have found this difficult to stick to, mostly because I am working on a medium-sized project that sometimes feels like progress is not being made at all. My remedy to this feeling has been to work on something that can be delivered in two days or less, alongside my main projects. So far, this has been effective. I just hope to remain consistent with it until the end.

So far here are the things I have been working on:

- ClickTSDB: a time-series database designed to operate standalone or act like [Prometheus remote storage](https://prometheus.io/docs/prometheus/latest/storage/#remote-storage-integrations). It's backed by ClickHouse database and being developed in Rust. I have been able to implement.
    - Prometheus remote read/write protocol
    - A lightweight full-text search library: to provide fast indexing of time-series labels in front of ClickHouse. Indexing inside ClickHouse made the data modeling hard and was not taking advantage of ClickHouse performance. Most importantly, it wouldn't have been fun as it would just be like packaging a bunch of tools. Let's see how far this goes. 

- [Facebook gorilla paper](https://www.vldb.org/pvldb/vol8/p1816-teller.pdf): Read and implemented in Zig the Facebook time-series compression algorithm. Yes Zig!!, The plan is to become better at Rust, Golang and Zig during this batch. You can find the code [here](https://github.com/evanxg852000/gorilla). I will also be glad to see it reviewed by fellow ziguanas.

- [Rust atomics and locks](https://marabos.nl/atomics/): I have read and coded along this great book written by Mara Bos. I even went on to try an implementation of Golang WaitGroup in Rust using [mutex](https://gist.github.com/evanxg852000/af3212ebc1a0d7b1645fb594a43abcfa) and [atomic](https://gist.github.com/evanxg852000/fba7350c09717aa8ff68af20e7cc043f)


While this might seem productive, there are areas where I'm falling behind:
- [rust-camp](https://github.com/rust-lang-ua/rustcamp): I'm still stuck at chapter 2, but I'm determined to finish it as I have deeply understood few more Rust concepts. 
- [Building Git](https://shop.jcoglan.com/building-git/): I initially joined a study group for this during the first week, intending to code along. However, I'm still on chapter 3.

These are tasks I added to my plan when the batch started. As you can attest, no plan remains unaltered at the contact of ~~enemies~~ passionate fellow developers. The recursers have a solution in the form of advice: when in doubt, just write code.

In summary, my time at the Recurse Center has been both busy and enjoyable, and I encourage anyone who can to apply. Based on my experience, including the application process, it's evident that passion and the ability to work empathically with others are the key factors at the Recurse Center. While I don't plan to write regularly due to the workload, I'll document as much as possible here. Additionally, I may attempt to video record demos of the projects I'm working on and publish them without further edits. In the meantime, let me get back to coding.

Thanks for reading 👋🏾
