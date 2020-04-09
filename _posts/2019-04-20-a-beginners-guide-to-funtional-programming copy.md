---
layout: post
title: "A beginners guide to functional programming"
date: 2019-04-20 06:30:05
categories: [tutorial]
comments: true
---

Hello, in today's post, we will explore functional programming along with some techniques and examples.

<!--more-->

![](https://drive.google.com/uc?id=1gc5Mzer8Ly_G_RHwZ1-uX6vL5elP3Q6f)

**Note**:  Before we dive in, I would like to stress that ***functional programming is not a silver bullet.***

According to wikipedia: 
> Functional programming is a programming paradigm, a style of building the structure and elements of computer programs, that treats computation as the evaluation of mathematical functions and avoids changing-state and mutable data.

Sounds like a lot going in this definition. In general, we have commonly two majors hight level programming paradigms: 
* Imperative programming: in which the developer instructs how to perform a certain task.
* Declarative programming: in which the developer list a set of properties of the desired result.

In imperative programming, the programmer mostly changes the state of the computing environment via statements. In this paradigm, you will mostly encounter procedural programming and object-oriented programming. Whereas in declarative programming, the programmer merely describes the desired result as opposed to instructing how to achieve it. In this paradigm, you find logic programming and functional programming. The later which is the subject of this post.

Now let's discuss some principles.
Object-oriented programming comes with the following principles: abstraction, encapsulation, polymorphism, and inheritance. On the other hand, functional programming comes with a set of principles among which immutability is the most important. In fact, some object-oriented features can be achieved in functional programming with ease. The later comes with the added benefit of being able to reason about the code and easily trace errors. The programming units in functional programming are called well...  😃 Functions.

Mutability can cause a certain type of bugs that are difficult to trace, sometimes hard to reproduce in the worse case: race condition bugs encountered in concurrent programming is one such category of bugs.

The immutability rule states that programs and  functions should not mutate the environment:
* Variables should not be reassigned.
* Functions should be pure: without side effects.
* Functions should be referentially transparent.

In mathematical term and more generally, a function is meant to return a value to the caller (the environment). Anything else it does to the environment aside from that is called side effect. Following is an example of a function with side effects also called **impure**. The function `add` mutates the global variable `counter`. 

```python
counter = 0;

def add(a, b):
    global counter
    counter += 1  # side effect
    return a + b

print(add(3,4)) # outputs 7
print(counter) # outputs 1
```

Referential transparency requires a functional computing unit (function) to always return the same value when passed the same parameter. Although one might easily relate one to the other, It has to be stressed that purity of a function has nothing to do with its referential transparency. The following `addTo` function is pure but not referentially transparent.  It depends on the external environment to compute its result. Thus `addTo` computation can change based on external state.

```python
base = 0;

def addTo(a):
    global base
    return a + base

print(addTo(3)) # outputs 3
base = 2;
print(addTo(3)) # outputs 5
```

Functional programming can only be practiced in languages that provide  some set of features:
 
* Functions must be first class objects: meaning functions should be treated as regular variables/values in the language. Assigning a function to a variable, passing a function to another function and returning a function from another should be supported operations.
* Expressions must be supported: It might seem obvious, but a language made solely with statements cannot be functional. Because most programming languages nowadays are multi-paradigm, this can easily be taken for granted.  A DSL for driving a bot is a good candidate for a language only made with statements.  
* To make it convenient for developers in passing functions around; functional languages often implement lambda expressions.

Now that we have some basics understanding of functional programming, let's discuss some techniques along with examples.
Recursion is at the heart of functional programming, we won't dive into explaining recursion in this post. But in functional programming, recursions are preferred over loops. In fact, loops are not allowed as mutations happen in loops. Loops exist in languages that are imperative. 
You might wonder 🤔 how can one iterate over a collection? Alternatives are provided as function in order to process collections. In multi-paradigm languages  (python, javascript etc.), these functions can be implemented iteratively via loops since they already provide these constructs. But in a purely functional language such as Scheme, walking a collection is implemented recursively. In fact, at the core of these implementations exist in most cases a recursive data structure.

In the comming examples, we will build a list as a recursive data structure and implement two of the most important list processing functions.

```python
def pair(a, b):
    return (a, b)

def first(p):
    return p[0]

def second(p):
    return p[1]

my_list = pair(2, pair(4, pair(3, None))) # (2, (4, (3, None)))
first(my_list) 		# yields -> 2
second(my_list) 	# yields -> (4, (3, None))
```

In the previous snippet, we have defined a `pair` function that can make a data structure to hold two elements. For the sake of simplicity, we used python tuples. One functional purist might use a closure instead of a tuple. We also defined `first` and `second` functions to help us access the components of our pair. Constructing a list is just as easy as embedding pair within another pair.

Now that we have a data structure, let's explore how we could implement `map` and `fold` for a collection. Note that `fold` is also referred to as `reduce`.

```python
def map(lst, fn):
    def _map(ilst):
        if ilst is None:
            return None
        return pair(fn(first(ilst)), _map(second(ilst)))

    return _map(lst)
    
def fold(lst, fn, acc):
    if lst is None:
        return acc;
    return fold(second(lst), fn, fn(acc, first(lst)))

map(my_list, lambda x: x * x)    # yields -> (4, (16, (9, None)))
fold(my_list, lambda acc, cur: acc + cur, 0)    # yields -> 9
``` 
The `map` function recurse down our list until it hits the sentinel value `None` from where it starts constructing and returning the pairs from the inside while also applying the function `fn`. We first define a recursive helper function `_map` that first check if the list passed in is `None`. This check represents the base case of the recursion. Then we return a constructed pair. The first entry is obtained by applying the function `fn` to the first entry of the list. The second entry is a recursive call to the inner `_map` function while passing the second entry of our list that could embed subsequent pairs. The following snippet shows how the list gets constructed on each stage of the recursive call.

```bash
# origianl list -> (2, (4, (3, None)))
(9, None)
(16, (9, None))
(4, (16, (9, None)))
```

The `fold` function, on the other hand, accepts an initial value as the third parameter. This parameter is used as the accumulator of the values as we recurse down the list. Once the recursion hits the base case, we return the value of the accumulator. This recursion has a specific shape that's worth mentioning. The temporary accumulator value is always handed over to the next recursive call. Therefore, to calculate the final result, we don't need to keep the previous recursive calls environment on the stack since all we needed from that environment is already passed as an argument. This pattern called [tail recursion](https://en.wikipedia.org/wiki/Recursion_(computer_science)#Tail-recursive_functions){:target="_blank"} can be recognized by most compilers or interpreters for optimization.

Moving into our functional programming adventure, I would like to discuss a concept referred to as [currying](https://en.wikipedia.org/wiki/Currying){:target="_blank"}.
It's the concept of converting a function that takes multiple arguments into a series of a function call that take a single argument. This mathematical concept was introduced by Gottlob Frege, and Moses Schönfinkel, and further developed by **Haskel Curry** (thus the name). 🤔... Yeah, you're right. He is the same guy whose name is also known from a very popular functional programming language [Haskel](https://www.haskell.org/){:target="_blank"}. The opposite of this process is obviously called uncurrying. 

The following is an example of two functions that will respectively curry and uncurry a function accepting two arguments. Although we are writing for a two arguments function, this can be rewritten for **N** arguments. Feel free to take it as a challenge.

```python
def curry(fn):
    return lambda x : lambda y: fn(x, y)

def uncurry(fn):
    return lambda x, y: fn(x)(y)

cadd = curry(lambda a, b : a+b) # curry a lambda that does a + b 
add = uncurry(cadd) # uncurry the previously curried lambda

cadd(3)(5)  # yields -> 8
add(3, 5)   # yields -> 8
```

We have come to the closing end of this adventure in functional programming 👏 👏 👏 🕺. I highly encourage you to explore a purely functional language (Haskel, Lisp). I also recommend you to implement a `filter` function for our recursive list data structure. Languages such as Scheme might bend your mind in the first place, but they will teach you a lot. While you might not move completely to a purely functional language, I believe functional programming will give you the skills to write concise and readable code in certain scenarios. Most importantly it will make you watch out for unnecessary mutation and side effects. This latter aspect will free your programs from certain types of icky bugs.
