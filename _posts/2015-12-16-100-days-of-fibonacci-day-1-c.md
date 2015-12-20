---
layout: post
title: 100 Days of Fibonacci - Day 1, C
video: false
comments: false
---

Yesterday I implemented the directly recursive and the tail recursive Fibonacci
algorithms in Haskell. Today I decided to take one of the most widely known
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

Overflows are something one needs to be careful about in C. C provides
very little abstraction compared to yesterdays language Haskell. In
Haskell we used a data type which supports unlimited integer values.
This means that the compiler links the code to a library that provides
a string representation of integers, along with operations for working
on integer strings.
This library can also be implemented in C, but it is out of scope for
the project.

The full code for Fibonacci in C is available 
[here](https://github.com/madsbuch/snippets/blob/master/fibonacci/fib.c).
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

