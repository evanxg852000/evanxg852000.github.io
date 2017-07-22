---
layout: post
title: "Creating namespace in Javascript"
date: 2013-12-27 02:09:00
categories: [tutorial]
comments: true
---

Structuring javascript code by using namespace is a great way to organise source code. However because Javascript does not have namespace built-in feature; it becomes verbous to create and use namespaces.One of the Javscript library that makes use of namespace heavily is [Qooxdoo](http://qooxdoo.org).

<!--more-->

In this Qooxdoo, you create an instance of button like this:  

{% highlight js %}
var button = new qx.ui.form.Button("Button");
{% endhighlight %}

First of all, lets create namespace in javascript. 

{% highlight js %}
//create an empty object that is the root of the namespace 
var com={}; 
//any other sub-namespace should be explicitly or implicity delcared as object
com.evansofts={};

com.evansofts.lib={
	hello: function(){
		console.log('Hello from com.evansofts.lib');
	}
}

com.evansofts.main={
	hello: function(){
		console.log('Hello from com.evansofts.main');
	}
}
{% endhighlight %}

{% highlight js %}
hello();//this won't work as hello is undefined on the global namesapce
{% endhighlight %}

As a side note, the global namespace is the `window` object. In order to call our functions we need to fully qualify their name like this:

{% highlight js %}
com.evansofts.lib.hello()
com.evansofts.main.hello();
{% endhighlight %}

As you can see we need to type the namespace all the way down to our function `com.evansofts.lib.*` in order to access it. Wouldn't be great to uses statement like inport or using as in java and C# respectively. well we can implements such utility function. Since javascript namespaces in the way it's defined are just objects, the idea is to attache these objects to the global namespace `window`. that way we can access all the properties of those objects from the global namespace. A simple solution will be to create a utility function like this:

{% highlight js %}
function using(object,as){
	window[as]=object;
}
{% endhighlight %}

Assuming you call the function like this `using(com.evansofts.lib, 'library')`, you can now do `library.hello()` in your code without any worries. You will notice that we have successfully bound the namespace. Now I would like to add that syntactic sugar to our utility function; I would like us to use it in this way `using(com.evansofts.lib).as('library')`. This syntaxe clearly explain the purpose of this function. So we will redefine our function like this:

{% highlight js %}
function using(object){
  this.as=function(alias){
    window[alias]=object;
  }
  return this;
}
{% endhighlight %}

This litle change will allow us to use the function like so:

{% highlight js %}
using(com.evansofts.lib).as('library');
library.hello();
com.evansofts.main.hello(); //notice the namespaces are still consistent
{% endhighlight %}

Some people will say well we still have to type `library.*` in order to access object of the namespace. Can we pull everything to global namespace? Yes is the answer. You will just have to loop through the namespace and add all its objects and properties to the global namespace wich is the window. However, if you plan doing that, then there is no point to create namespaces in you code as you will be removing the organisation you brought in your code by introducing namespaces.