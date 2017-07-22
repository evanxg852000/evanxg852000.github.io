---
layout: post
title: "The fibonacci series using memoization"
date: 2011-12-16 08:01:00
categories: [tutorial]
comments: true
---

Just another algorithm today...
This is the Fibonacci series implemented using memoization technique.

<!--more-->

{% highlight cpp %}
int fibonacci(int n){
  int prev=0;
  int next=1;
  int val=0;
  for(int i=2;i<=n;i++){
    val=next+prev;
    prev=next;
    next=val;
  }
  return val;
}
{% endhighlight %}

Typically, If you are designing a maths library, you would implement a caching mechanism to avoid reprocessing again. A better  way in term of performance would be:

{% highlight cpp %}
int fibonacci(int n){
  static vector<int> cache(0);
  //check if we have already processed
  if(n<cache.size()){
    cout<<"from cache: ";
    return cache.at(n);
  }

  //initial values 
  if(cache.size()==0){
    cache.push_back(0);
    cache.push_back(1);
  }

  //we calculate the fibonacci from where the cache end to the number requested
  for(int i=cache.size()-1;i<=n;i++){
    cache.push_back( cache.at(i) + cache.at(i-1) );
  }
  cout<<"Calculated : ";
  return cache.at(n);
}
{% endhighlight %}

*NB: the `cout` is embedded for simple debugging* 
