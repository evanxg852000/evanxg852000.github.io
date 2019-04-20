---
layout: post
title: "Let’s build a NodeJS micro framework (Part III)"
date: 2019-04-02 05:30:05
categories: [tutorial]
comments: true
---

*Learning to build an express like NodeJS framework …*

<!--more-->

![](https://drive.google.com/uc?id=12bGSq5lsB6huFTxGPZx_NXo8vlmuCD6k)

We covered the skeleton implementation of our framework in [Part I](https://evanxg852000.github.io/tutorial/2019/01/05/lets-build-a-nodejs-micro-framework-part-i.html). We also took care of the router implementation in [Part II](https://evanxg852000.github.io/tutorial/2019/01/12/lets-build-a-nodejs-micro-framework-part-ii.html). Today, we will be looking into how to implement the template engine. The template engine will be a basic compiler that will translate our template code into Javascript code. When a template needs to be rendered, the generated code is loaded and executed. Our template engine should avoid unnecessary template compilation; especially when running an application in a production environment.

Before we dive into how to implement the template engine, let's give an overview of the template syntax as well as the translated template source file. In practice, we usually want to have a layout file from which other template files can derive from. An example of such a layout file should look like the following snippet.

<script src="https://gist.github.com/evanxg852000/a7536836787f893dc934f65d299cb9ec.js"></script>

From the above snippet, we can already see two template syntax being introduced:

* {% raw %}`{{ title }}`{% endraw %} is the syntax for displaying the string interpretation of the variable title
* {% raw %}`{% block content %}`{% endraw %} denotes a block that acts like a placeholder to be overridden by child templates

The following is an example of a child snippet inheriting our main template file.

<script src="https://gist.github.com/evanxg852000/0e6eed405333f3c421efa0ce0bfbb498.js"></script>

As you may have already guessed, the first line tells the template compiler that this template inherits another template file called `main.html` located in the templates directory. The block syntax this time is intended for overriding blocks declared into the base template file. Just like overriding a method requires the signature to match, overriding a block also requires the name to match. Next, we have the `if` and `for` constructs. Luckily for us, the target language is Javascript. Thus, we can limit ourselves into understanding the higher level constructs rather than building a full parsing and type checking mechanism. We can just use Javascript as a superset of our template syntax. This means writing pure Javascript into part of our template should yield a syntactically valid template. An example of this is: `for (let it of items)`.

We will save the compiled version of the template into the `.tmp` subfolder. In this folder, template files will be named with a hashed version of the original template file name. Doing so will allow us to check if templates/.temp/5017803b9ee9b00cc52db4a18a64b71cfc076fd7.js exists when test.html is requested. If so, we load and execute the only function it exports. Otherwise, we compile the template from `templates/test.html`. This is how template changes get reflected automatically during the development cycle. This is also how we achieve template caching in production as templates sources don't change often in production. Therefore, no need to recompile the template from the source every time it's requested. Consequently, any update to a production instance of an application based on our framework will require users to delete the `.tmp` folder in order to invalidate the template cache.

The following is an example of a generated template file. Notice the way we named our variables inside the exported function `__njsOutput`. This is a technique related to code generation in order to avoid name collisions.

<script src="https://gist.github.com/evanxg852000/762f167fe74f00e9c5e89c44fa96f426.js"></script>

Now that we have a grasp of the template syntax, let's dive into the implementation details by first showing a skeleton of the template class. The implementation is longer than what we have seen so far in this series. Therefore, we will only show the most important parts of the source code. However, we will discuss all the internal methods as we go along.

<script src="https://gist.github.com/evanxg852000/f99ce6d16163e1b0c31522dff2d09164.js"></script>

Our template engine can essentially do two things: compile a template and render a template. In order to compile, it needs to tokenize the code, parse the tokens, and generate the target code. These steps are typically the most essential stages of any compiler/translator.

* Tokenization: also known as scanning or lexing is the process of taking source language and splitting it into most basic unit of the language. This is the equivalent of breaking a paragraph into words.
* Parsing: also known as semantic analysis is the process of understanding the structure of language by building an abstract syntax tree/semantic tree out of the tokens.
* Generation: is the process of generating the target language from what was understood -the abstract syntax tree.

Please note that in real compilers, there are other steps such as type checking and optimization which may correspond to grammatical checks, natural language translation, cultural context translation for English.

Now let's dive in some implementation details. Our first stop is the simple constructor.

<script src="https://gist.github.com/evanxg852000/f803feae6d0d7fea3dc769b6b4de6bdd.js"></script>

The constructor takes the directory where templates should be located, checks the existence of this directory and make sure subdirectory .tmp is also ready. It also accepts a boolean variable denoting if caching should be enabled. Recall that this depends on whether the app is running in production or development mode.

**Note:** *`existsSync` may not seem to you as an idiomatic way of doing things in NodeJS, but there are cases where synchronous actions are more appropriate.*

Now let's tackle the render method.

<script src="https://gist.github.com/evanxg852000/bdd199c18f88e9a6c7f848057b89cdd3.js"></script>

The `render` method will accept the template file to render as well as a context. A context is a Javascript object containing all data that will be available to the template file during rendering as variables. We first build the source file path and get the compiled file name by calling `this._compiledFile(file)`. If the template file is not found, we just raised an error. Next, we check if caching is enabled and if a precompiled template exists. If so, we require that file and renders it by passing the `context`. Otherwise, we need to compile the template: -always the case in dev mode. We first compile the template into an AST, then walk the AST to generate and save the final template file by calling `this._saveGenCode(file, this.generate(ast))`. Finally, we load and execute it as we did before for the precompiled one. Now let's explore the other methods.

<script src="https://gist.github.com/evanxg852000/3dd52d8b3121773128d0e31a109174d9.js"></script>

The compile `method` first reads the template file, tokenizes and parses the content to build the abstract syntax tree as a `TemplateNode` object. The `_tokenize` helper method splits the template source code into chunks that can be processed in the parse method. In the parse method, we create a Javascript object named `_parser` that will maintain the state of the parsing activities. The next step is to construct the root node of the AST represented by a TemplateNode. We then call the helper method `_parse` to build the whole AST recursively. This technique of parsing is called recursive descent parsing as the parse tree is built from top to down by recursively parsing child nodes.

<script src="https://gist.github.com/evanxg852000/9599125ab21757fe61d4ba4ce31f4a5f.js"></script>

So the `_parse` helper method takes the AST and an array or string denoting when to stop parsing the current construct. It loops through the tokens using the parser state. Tokens come in two flavors: {% raw %}`{{ code }}`{% endraw %} and {% raw %}`{% code %}`{% endraw %}. First, we get rid of the markers and split the code into expressions array. Next, we check whether to stop or keep moving. Then comes a set of checks to match which construct should be parsed. If the token starts with {% raw %}`{{`{% endraw %}, we parse an OutNode for output. Regarding other cases, we need to extract the first expression inside {% raw %}`{% code %}`{% endraw %}. Since we already have code split into expression array, the first entry should be the keyword expressing the construct we are dealing with. We assign that first entry to `keyword` and check against `extends`, `block`, `if`, `for` to parse the construct. If none matches then we build a default TextNode that will just output the raw template text. For brevity reasons, we won't go over the implementation details of the different Nodes, Please refer to the repository on [Github](https://github.com/evanxg852000/njs-micro){:target="_blank"}.

The last snippet we will show is the code generation method. It simply walks the tree and calls `generate` on each Node. Just like the parsing method, every Node knows how to generate its target code with the assumption that the following variables will be available: `context`, `__njsOutput`.

<script src="https://gist.github.com/evanxg852000/a8a03f5150f117af0792df7eb75ea323.js"></script>

If you made this far by coding along, please give yourself a standing ovation 👏 👏 👏 🕺. Well, fake it! as if you just broke a long-standing sporting record.

I hope you enjoyed reading as much as I did writing this series while leveling up my writing skills. Now rather than using this as a basis to create the Nth framework, please use this as a basis to contribute to production ready frameworks. Thanks for reading. Your comments, suggestions, and pull requests are most welcome on [Github](https://github.com/evanxg852000/njs-micro){:target="_blank"}.
