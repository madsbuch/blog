---
layout: post
title: 100 Days of Fibonacci - Day 2, Java
video: false
comments: false
---
This is an article in a series of articles. An overview of the entire
project can be found [here](/blog/100-days-of-fibonacci-overview/).

So far I have used Haskell, a pure functional language, and C,
an empiric systems language. In these languages I have implemented
the Fibonacci function recursively, recursively with an accumulator,
and iteratively.
There are still some basic techniques left, so today I will not
introduce something radical language-wise. Instead I will take one of
the most widespread languages and implement Fibonacci by dynamic
programming using memoization.

# Day 2 - Java
Here I have implemented Fibonacci in Java using dynamic programming. The
idea is to check whether the result has already been calculated. If
it has so, then we return the already calculated the result, otherwise
we calculate the result, save it, and return.

{% highlight java %}
/**
 * Memoization implementation of the Fibonacci function using hash table
 * as container for memory entries
 */
private static BigInteger fibDynamic(int n, Hashtable<Integer, BigInteger> mem){
    // Check if we have the result
    if(mem.containsKey(new Integer(n)))
        return mem.get(new Integer(n));

    // Base cases
    if(n == 0)
        return new BigInteger("0");
    if(n == 1)
        return new BigInteger("1");

    // calculate and memoize Fibonacci
    mem.put(new Integer(n), fibDynamic(n-1, mem).add(fibDynamic(n-2, mem)));
    return mem.get(n);
}
{% endhighlight %}

To keep the type signature to that of the other implementations, we build an 
overloaded wrapper method, which initializes a hash table, runs the
recursive function and returns its result.

{% highlight java %}
/**
 * Driver for the dynamic implementation of Fibonacci
 */
private static BigInteger fibDynamic(int n){
    Hashtable<Integer, BigInteger> mem = new Hashtable<Integer, BigInteger>(n+1);
    return fibDynamic(n, mem);
}
{% endhighlight %}

The code is as usual available and can be found
[here](https://github.com/madsbuch/snippets/blob/master/fibonacci/Fib.java). 

# Fibonacci and Dynamic Programming
Today I introduced dynamic programming. We have two ways to apply dynamic
programming, either by memoization or tabulation. Memoization is the process
of remembering intermediate results. This is exactly what I did in the
earlier example. Here we might cache some results we never use.
Tabulation is the process of pre-calculating a table of intermediate results
and then afterwards aggregate the result afterwards.

Dynamic programming can also be seen as a method of reducing the number
of unfolds. In this example the Fibonacci implementation is called linear
number of times to _n_. In the direct recursive method, it would
have been exponentially in _n_.

# Java, Java, Java
Java is one of the top languages used today. It is compiled to an
intermediate representation, Java byte code, before it is ran on a Java
virtual machine. This makes it very portable without sharing the Java
source code. Furthermore there is a big body of libraries and articles
written solving various problems in the language.

When having an enterprise it is certainly also a big plus that
most programmers program in Java. It is easy to find people who
are able to read and write Java code.

