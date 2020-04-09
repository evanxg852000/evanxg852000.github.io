---
layout: post
title: "Inverted Index: a simple yet powerful data structure"
date: 2020-04-09 08:30:05
categories: [tutorial, rust, data, structure]
comments: true
---

Hi, it’s been a while since I last wrote an article. It’s a really strange time as our world is battling the #COVID-19 Pandemic. Despite all attention to this pandemic, I would like to discuss in this post an important data structure named Inverted Index. I also hope to make the next five minutes of your confinement useful while doing so.

<!--more-->

What is an inverted index I hear you asking? An inverted index is a data structure that supports full-text search. It does this by storing mapping content chunks (words or numbers) also called terms to document name or location along with metadata such as frequencies, position, etc…
This data structure is so important in full-text search that it constitutes today the base structure for most popular full-text search engines: Elastic search,  Apache Solr etc.

In term of structure, an inverted index is just a hash table in which keys corresponds to chunks from a set of documents, and values correspond to a list of documents in which that chunk appear.
In search terminology:

-  A chunk is called `term`
- A document is called `corpus `
- The set of documents is called `corpora`
- The database (inverted index) supporting the search is a  called `search index`
- The process of adding a document to the search index is called `indexing`

![](https://drive.google.com/uc?id=16KKjzjMavO3gYgmoaI0jCXkOktUDEnh0)

The preceding picture is a basic representation of the inverted index along with the indexing process. In practice, not all terms will make it through to the index table. Terms like stop words (a, and, to etc…) or plurals are not relevant to searching text documents. Therefore, some terms are left out during the indexing process depending on the context. In fact, there is a whole pre-processing and filtering step through which terms go before having the chance to land in the index. Please read on information retrieval ([1](https://nlp.stanford.edu/IR-book/information-retrieval-book.html){:target="_blank"}, [2](https://en.wikipedia.org/wiki/Stemming){:target="_blank"}) if you want to deep dive in this topic.

Now that we have scratched the surface of search technology and learned a bit about the inverted index data structure, we can now turn our attention to a basic implementation. For the purpose of this post, I have chosen to implement it in [Rust](https://www.rust-lang.org/){:target="_blank"}. 🙌 Yes! this is my first post that contains rust code; the most loved programming language of the past four years.

To start, we will create a `struct` and implement a `new` method to initialize the three fields. 
- `count` will represent the number of documents in this index, it also serves to assign a basic unique id to the docs
- `docs` is a HashMap that will match document `ids` to document names
- `index` is also a HashMap that will match terms to another HashMap of document ids to term frequencies.
We store the term's frequencies in order to demonstrate basic search result ranking. This is not the state of the art ranking algorithm ⚠️.

<script src="https://gist.github.com/evanxg852000/af4ec1ffbc80dfe3c047a17edd8b4444.js"></script>

Coming next is the implementation of both `add_doucment` and `search` methods. In reference to the pre-processing and filtering step, here we basically convert each term to its lowercase version before adding to the index. conversely, when searching we need to make sure the search term is also in lowercase.

<script src="https://gist.github.com/evanxg852000/0ee2972315dde53d14c4f74af8644f9f.js"></script>

That's a pretty simple version of an in-memory full-text search engine. As you can see, it's really simple to implement. The most difficult parts of a search engine are usually:  document pre-processing,  index compression and search result ranking. You can play with the full source code here  on [Rust Playgroung](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=638cf94bef8a250c759965acf1b86886){:target="_blank"} or [Github](https://github.com/evanxg852000/blog-codes/tree/master/rust/inverted-index){:target="_blank"}

I hope this short post taught you something interesting and until next time, stay home - stay safe.
May God bless & protect us all 🙏.