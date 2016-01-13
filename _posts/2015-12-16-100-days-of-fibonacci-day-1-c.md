---
layout: post
title: 100 Days of Fibonacci - Day 1, C
video: false
comments: false
---

My [100 days of Fibonacci](/blog/100-days-of-fibonacci-overview/) challenge
started yesterday where I implemented the function directly recursive
and using accumulated recursion in Haskell.
Today I decided to take one of the most widely known
languages and implement of iterative edition of the Fibonacci function.

# Day 1 - C
In C I implemented Fibonacci iteratively. In the implementation I have used 
the type `long` to store intermediate results and return values. It
guarantees 32 bit of storage, which allows us to compute large enough results.

{% highlight c %}
unsigned long fib_iterative(long n){
    unsigned long i=0, a=0, b=1;
    for(i=1 ; i<=n ; i++){
        unsigned long tmp = a+b;
        a = b;
        b=tmp;
    }
    return a;
}
{% endhighlight %}

Overflows needs to be carefully thought about in C. The language
provides very little support for unlimited integers compared to
yesterdays language, Haskell, which indeed had support for big integers.

Haskell supports large integers by using a library which uses strings
for representing integers along with implementations of arithmetic
functions for working on these integers. This could also have been
implemented in C. However, this is out of scope for this article.

The full code for Fibonacci in C is available 
[on Github](https://github.com/madsbuch/fibonacci/tree/master/c).
Be aware that the code might change when I feel for implementing other
algorithms in C.

# Iterative Fibonacci
The iterative Fibonacci has a linear time complexity. This is the same
as the tail recursive implementation. These two implementations also
have much in common.

The iterative implementation con not be implemented in pure
functional languages because it requires mutable states.
This is seen when we change the value of `a` in the
above example by `a = b;`. Furthermore the _while_ concept is not natural to
functional programming, where operations on collections of elements
are done recursively.

# C Programming Language
C was meant to be close to the hardware. As a slight layer
on assembly programming. A language used for programming operating
systems and hardware driver. And so it is fast as it has a minimal
runtime system and lets the programmer have complete control over
the hardware.

The control is also often the downside of this language. As it provides
a limited set of abstraction, it is easy to make errors. Errors like
buffer overflows are not rare when looking at projects written in C.

