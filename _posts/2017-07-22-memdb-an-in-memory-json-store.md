---
layout: post
title: "MemDB; simple, fast and pure Javascript in memory database for Node JS."
date: 2017-07-22 18:49:00
categories: [misc]
comments: true
---

This lightweight engine can be used in place of things like LevelDB, Redis while keeping things light and very simple. Internaly, memDB uses json as storage format with an opt-in support for encryption.

<!--more-->

 In order to achieve performance, memdb keeps things in a staging area for fast access. No worries, every booking is handled for you :). We are developers too, thus we hate keeping tract of things.

Quick start
--------

{% highlight js %}
const MemDB = require('memdb')
const db = new MemDB('_starwars', {encryptionKey: 'secret'}) 

db.put('skywalker', {
    force: 'light',
    name: 'Luke Skywalker'
}).then((status) => {
  console.log(status) // => true
})

db.get('skywalker').then((rsp) => {
  console.log(rps) // => [Object skywalker...]
})
{% endhighlight %}

MemDB's API is pretty simple and straigth forward. all async action api return a promise.

- `version()`: Get MemDB current version
- `revision()`: Get the current database revision
- `options()`: Get the database instance options
- `put(keychain, value, loose)`: Save data in the database
- `get(keychain, defaultValue)`: Get data from the database
- `all()`: Get all data fron the database
- `delete(keychain)`: Delete data from the database

NB: What the heck is keychain ?
keychain is a concatenation of multiple object keys to obtain a path for acessing an object property in a deep level. This is a convenient way of puting/getting deeply nestes object properties. The folowing keychain `server.dev.host` means accessing the `host` property form `dev` object which is in turn available in the `server` object as a property.

For more information, please head over [MemDB](https://github.com/evanxg852000/memdb)