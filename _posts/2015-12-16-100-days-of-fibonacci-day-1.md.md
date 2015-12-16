---
layout: post
title: 100 Days of Fibonacci - Day 1
video: false
comments: false
---

Yesterday I implemented the directly recursive and the tail recursive Fibonacci
algorithms in Haskell. Today I decided to take one of the most widely known
languages and implement of iterative edition of the Fibonacci function.

# Day 1 / C
In C I implemented Fibonacci iteratively. In the implementation I have used 
the type `long` to store intermediate results and return values. It
guarantees 32 bit of storage, which makes good room for overflow.

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

Overflows is something one needs to be careful about in C. C provides
very little abstraction compared the yesterdays language Haskell. In
Haskell we used a data type that supported unlimited integer values.
This means that the compiler links the code to a library that provides
a string representation of integers, along with operations. This library
can also be implemented in C, but is not as easily accessible.

The full code for Fibonacci in C is available 
[here](https://github.com/madsbuch/snippets/blob/master/fibonacci/fib.c).
Be aware that the code might change when I feel for implementing other
algorithms in C.

# Iterative Fibonacci
The iterative Fibonacci has a linear time complexity. This is the same
as the tail recursive implementation. These two implementations also
has much in common, which is evident when implementing both.

The iterative implementation is not an opportunity in pure functional languages
as it required mutable variables. Note how we change the value of `a` in the
above example by `a = b;`. Furthermore the _while_ concept is not natural to
functional programming.

# C Programming Language
C was a language meant to be close to the hardware. As a slight layer
on assembly programming. A language used for programming operating
systems and hardware driver. And so, it is fast. It has a minimal
runtime system and lets the programmer have complete control over
the hardware.

The control is also often the downside of this language. As it provides
a limited set of abstraction, it is easy to make errors. Errors like
buffer overflows are not rare when looking at projects written in C.

