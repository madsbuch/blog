---
layout: post
title: 100 Days of Fibonacci - Day 3, Scala
video: false
comments: false
---

Coming from yesterdays Java, Scala is a natural follower. It is
a functional language compiled to run on the Java Virtual Machine.
It integrates easily with Java programs, and such, it is often used
in conjunction with Java. Javas API is also available in Scala. This
makes it easier for Java developers to start with this language if they
want to dive into functional programming.

# Day 3 - Scala
Today I present an implementation of Fibonacci in continuation passing
style (CPS). This can easily be done in Scala because of its functional
nature:

{% highlight scala %}
// Fibonacci in Contiuation Passing Style
def fib_cps (n: Int, k: (Int => Int)) : Int = n match{
    case 0 => k(0)
    case 1 => k(1)
    case _ => fib_cps(n-1, (a: Int) =>
        fib_cps(n-2, (b: Int) =>
            k(a+b)))
}
{% endhighlight %}

In this example I have used pattern matching to keep
it easily readable.
The variable _k_ holds the continuation which is evaluated
in the base cases (`case 0` and `case 1`).

The above code was wrapped to make it compatible to the
usually type signature of Fibonacci.

{% highlight scala %}
// An alias to make it satisfy the type signature Int -> Int
def fib_cont(n: Int) = fib_cps(n, ((x: Int) => x)) 
{% endhighlight %}

The main function holds a call to above. The whole thing is compiled
and run on the command line completely as one would expect coming
from Java.

{% highlight bash %}
$ scalac Fib.scala 
$ scala Fib
55
{% endhighlight %}

A little riddle: Guess $n$ such that $fib(n) = 55$.

# Continuation Passing Style
Continuation passing style is not easy to read, and such it is not a good way
to implement functions, unless there is a very good reason. These reasons
can be applications where explicit management of the control flow is
necessary.

Many compilers do an implicit conversion from recursive functions to
functions in CPS. The reason is that all computations are wrapped in a
package (i.e. the _continuation_) and is put in the heap which reduces
maximum stack size.

# Conclusion
Today I looked at 2 new things, Scala and continuation passing style.

Scala is a great language for writing functional code that has to
integrate with Java projects, or for writing functional code using
Java libraries.

Continuation passing style is a way of programming which can be enormously
useful. Especially when writing large project where the control flow has
to be explicitly managed.

