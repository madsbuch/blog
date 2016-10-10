---
layout: post
title: 100 Days of Fibonacci - Day 11, Fixed Point
video: false
comments: false
---

In this article we will look into encoding the Fibonacci function using the
`fix`-point combinator. This is an interesting function as it can be used
to implement general recursion in a programming language.

To set a context we first need to discuss some programming language
fundamentals: In daily speech programmers are used to programming
languages being express _all possible_ computations. This is, they are
[Turing complete](https://en.wikipedia.org/wiki/Turing_completeness).

However, often we want a programming language to support various properties.
Some often discussed properties are
[type safety](https://en.wikipedia.org/wiki/Type_safety), which entails that
programs that compile also runs without errors, and
[normalization](https://en.wikipedia.org/wiki/Normalization_property), which
roughly says we always get a result.

An often used framework for reasoning about these properties is the
simply typed lambda calculus (STLC) with friends. With the raw STLC
it is not possible to encode general recursion. Hence we need to add
a friend, which makes it possible.

This friend could be the fixed point operator.

# Day 11 - Fix 
Generally the `fix` operator looks as follow:

$$
    y \ f = f \ y \ f
$$ 

In this article we let Haskell be our syntactical point of view. This is
because we then will have an interpreter for free.

In the `ghci` interpreter we may define the function as follows:

```haskell
> let fix f = let x = f x in x
```

We now have a function `fix` that takes a function as argument, applies it on
itself and returns the result. If the function we apply `fix` on returns a
value, fix will return a value:

```haskell
> fix (\rec -> "Hello Fix!")
"Hello fix!"
```

Next, if we just apply the function on itself we will get an infinite loop:

```haskell
> fix (\rec -> rec)
^CInterrupted.
```

Now, this makes sense, as we continuously attempt to apply the lambda function
on itself.

In above example we only have a single argument, `n`, which is the function
itself. We can say that we have the recursion directly in our hands. But to
be able to do something interesting, like calculating the _i_'th Fibonacci
number, we need to add an argument we can decrement on. First a simple
example.

```haskell
> fix (\rec n -> if n == 0 then 0 else n + rec (n-1)) 10
55
```

In above example we use the `if` to return a value when we hit 0, otherwise
we apply the function on _n-1_. The expression simply adds all numbers from
0 to 10.

From here it is straight forward to implement the Fibonacci function. We need
to have a nested if-statement, as we have two base cases. Otherwise it is very
similar to the function above.

```haskell
> let fib = fix (\rec n -> 
    if n == 0 then 0 else if n == 1 then 1 else rec (n-1) + rec (n-2))
> fib 10
55
```

# Reflections
This is a directly recursive implementation of the Fibonacci function. This is
indeed visible from time complexity, and it is not feasible to compute the
Fibonacci number of large numbers. It is indeed possible to augment above
implementation with an accumulator.

Above implementation is interesting as we completely separate the recursion
from the actual computation. In
[an earlier article](/blog/100-days-of-fibonacci-day-0-haskell/)
I implemented the function in Haskell using normal function syntax. Here the
distinction between the recursion mechanism and the calculating mechanism is
not as distinct.

The separation is especially interesting when formally reasoning about
programming languages as we can get clutter away. This, however, is much
easier when doing this in dedicated languages.

Above examples are kind of futile in Haskell, as it already supports better
functionality for solving the Fibonacci function. It is, however, interesting
to think about what a _minimal_ language for solving the function would be.
In above implementation we need support for `fix`, `if-then-else`, boolean
constructions, and natural numbers. This is four language constructions.

