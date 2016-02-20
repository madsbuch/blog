---
layout: post
title: 100 Days of Fibonacci - Day 10, Python
video: false
comments: false
---

<!--{ Introduction to the project | }-->
It is yet another time for a post in my
[100 days of Fibonacci](/blog/100-days-of-fibonacci-overview) challenge.
Today I am containing myself a bit and provide a link between
a concept in programming and its root in mathematics.

<!--{ Introduction to Python | }-->
The concept is list comprehension in Python. Python is a widely
used programming language especially used as an easy language
appropriate for beginners. The languages is in particular
close to English. Furthermore it enforces good style by scoping by indentation.

<!--{ My take on Fibonacci in python }-->
As mentioned I zoom in on the list comprehension feature in
Python. I relate this feature with the way sets are defined in
math. Lastly I talk a bit about the interpretation of infinite sets in
programming.

# Fibonacci in Python
<!--{ Overview on the Python implementation | }-->
The implementation is the direct recursive implementation accompanied
by dynamic programming by momoization. This as
my [Java](/blog/100-days-of-fibonacci-day-2-java/) implementation. It is not
new to implement it like this, and simply provides a predicate to use in
the list comprehension.

{% highlight cs %}
cache = {}
def fib(n):
    if cache.get(n, None) != None :
        return cache[n]

    if n == 0 :
        return 0
    if n == 1 :
        return 1

    cache[n] = fib(n-1) + fib(n-2)
    return cache[n]
{% endhighlight %}

<!--{ Justification of the cache | }-->
The cache is here used as a global variable. Global variables are something one
should by all means avoid, but here the amount of code is small and the main
goal of this post is not software architecture.

<!--{ Transition to the list comprehension formulation |  }-->
From here we can know create the list of the 20 first Fibonacci numbers using
list comprehension in Python. We print it directly to provide an output.

{% highlight cs %}
print [fib(x) for x in range(20)]
{% endhighlight %}

This is a one-liner for mapping the list of numbers from 0 to 19 to the list
their corresponding Fibonacci numbers.

{% highlight bash %}
$ python fib.py 
[0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, ..., 4181]
{% endhighlight %}

The implementation is available from my supporting
[Git repository](https://github.com/madsbuch/fibonacci/tree/master/python).

# List Comprehension
<!--{ Set initial intuition in a for loop | }-->
The notation used above looks mostly like a very compact `for`-loop. For this
relation to work we need to slightly open up our understanding of lists and
iteration in programming languages.

<!--{ The new understanding of lists | }-->
First we need to view a list as a set. It should be quite clear that a list
indeed can be a set. For that to work we need operations like _intersection_,
_union_, _difference_, etc. Any programmer should be able to quickly implement
those. The contrary, that lists are sets, are not given. Lists are inherently
ordered, where sets are unordered.

<!--{ New understanding of iteration | }-->
The other thing is the `for`-loop. `for`-loops are usually viewed as iterating
all elements in a list one-by-one, and doing something to them. Instead we 
can see it as quantification. For a second we should forget everything about
the order of the list and details about iterations, and just see it as we 
perform an operation on every single element in the _set_.

We now have the mindset set for the following mathematical interpretation of
the above list comprehension to work. The following is a direct translation. 

$$
    \{ \ fib(x) \ | \ x \in \{0, 1, 2, 3, 4, 5, ..., 19\} \ \}
$$

which would usually be written as follows.

$$
    \{ \ fib(x) \ | \ x \in \mathbb{N} \ \}
$$

The reason why the last formulation is kind of futile is that Python
uses strict evaluation: All terms are evaluated. The set of naturals is
infinite and hence the computation will not halt.

The last formulation is, however, indeed possible to formulate in programming.
We just need a lazy evaluated language. As such we can formulate it, and use it,
in the Haskell programming language. Following works given we have an
implementation of the `fib` function:

{% highlight haskell %}
[fib n | n <- [0..]]
{% endhighlight %}

And what is all this worth? Well, a lot of material programmers have to
implement was first formulated in a mathematical language. This relations seeks
the demystify the relation between programming and math. In fact it can (and
is / will be in other posts) be argued that math and programming is in fact
the same thing. But for some reason the two areas are somewhat disjoint. This,
in my opinion, largely because of two cultures in communicating.

# Conclusion
<!--{ Fibonacci was implemented | }-->
In this post i implemented Fibonacci in python. There is nothing new to
the implementation. It was a directly recursive implementation supported
by a cache for speed.

<!--{ Conclusion of list comprehension | }-->
The concept elaborated in this post is list comprehension and its relationship
the the mathematical formulation of sets. Ultimately my message is that we as
programmers should be able to both formulate and interpret in other frameworks.
Also those seemingly very distant frameworks, like mathematics is for
many programmers.