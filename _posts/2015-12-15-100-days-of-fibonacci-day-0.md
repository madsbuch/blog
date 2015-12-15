---
layout: post
title: 100 Days of Fibonacci - Day 0
video: false
comments: false
---

Today I had my computer architecture class for the last time.
My students and I discussed the complexity of the various Fibonacci
algorithms. Fibonacci is a function often used to test out new
programming languages as it is easy to implement and can show
showcase many techniques.

Later on I worked with my friend
[Anders Schmidt](http://www.andersschmidt.com/) who showed me 
Paul Favius [100 Days UI Challenge](http://www.100daysui.com/).
I thought it was cool and wanted to do something similar, but
I do not design. So I decided to do it with Fibonacci
in 100 different programming languages. The idea is to showcase
some typical aspects of a programming language and implement the
function in language idiomatic ways.

# Day 0 - Haskell!
what else to start with?

In Haskell I have done two different implementations. First the
recursive way as we should usually think problems in a functional
context. We use pattern matching to pick out the base cases and
the induction case.

{% highlight haskell %}
-- Standard recursive implementation, very slow
fib :: Integer -> Integer
fib 0 = 0
fib 1 = 1
fib n = fib (n - 1) + fib (n - 2)
{% endhighlight %}

However, it turns out that this solution is quite slow.
Luckily there is advice! So I implemented it the tail-recursive
way also

{% highlight haskell %}
-- Using an accumulator
fibAcc :: Integer -> Integer
fibAcc n = doFib n (0,1)
    where
        doFib :: Integer -> (Integer, Integer) -> Integer
        doFib 0 (a, b) = a
        doFib n (a, b) = doFib (n-1) (a+b, a)
{% endhighlight %}

To keep the type signature identical to the other Fibonacci
we uses the `ẁhere` construction.

Lastly we need to make a command line program that takes the number, _n_,
we wanna calculate _fib(n)_ for. As Haskell is pure and lazy, we
can't simply in line system calls like we do in c. In Haskell a monadic
style is chosen.

{% highlight haskell %}
-- First argument is read and parsed as Integer
main = do
    a <- getArgs
    putStrLn $ show (fibAcc $ read (a !! 0) )
{% endhighlight %}

In the above example we first read the list of arguments into
`a`, where after we parse the first (0th) element and calculate
the corresponding Fibonacci number. The result is converted
to a string and written back to the command line.

The file is available for download
[here](https://github.com/madsbuch/snippets/blob/master/fibonacci/Fib.hs). 

# Functional Fibonacci
In this first implementation we used two techniques for implementing
the function

* __Recursion:__ The typical way of thinking of problems in a
  functional context. In this case it has a exponential
  time complexity. This is bad, as we know that the Fibonacci
  function can be implemented faster.
* __Tail Recursion:__ The idea is that we only keep track of
  the last stack frame. For Fibonacci we only care about the two
  previous results. This approach lets us, by an accumulator,
  forget all intermediate results and only pass forward the
  data we need. For Fibonacci we achieve a linear time complexity.

