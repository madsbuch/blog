---
layout: post
title: 100 Days of Fibonacci - Day 9, Haskell Types
video: false
comments: false
---

Haskell has a flexible type system. It actually is Turing complete
given the right language extensions. This also means that we can do
arbitrary computations, which we are going to exploit in this 10th 
day in my [100 days of Fibonacci](/blog/100-days-of-fibonacci-overview/)
challenge.

I already did [look at Haskell](/blog/100-days-of-fibonacci-day-0-haskell/).
So strictly speaking I should choose another language.
However, I find that programming in Haskell's type system is different
enough that I will consider it a different language.

# Day 9 - Haskell Types
Today I implemented Fibonacci in the Haskell type system. That means that I
can get the Haskell compiler to generate a _type_ for the _n_'th
Fibonacci number.

I first created a datatype for representing naturals.
The approach as in the last part of the
[Prolog implementation](/blog/100-days-of-fibonacci-day-4-prolog/) and
use used in [Coq](/blog/100-days-of-fibonacci-day-7-coq/) was used.
This representation
builds on the [Peano axioms](https://en.wikipedia.org/wiki/Peano_axioms)
and is straight forward to implement.

{% highlight haskell %} 
data Nat = Z | S Nat
{% endhighlight %}

This datatype is usable as a [kind](https://wiki.haskell.org/Kind)
because we use the language extension _DataKinds_. A kind can be
thought of as a type of a type.

In `ghci` we can inspect the kind of the type constructor `Z` by issuing
`:kind Z`. The result returned is `Z :: Nat`.

Next I implemented addition. In Haskell we use the
language extension _TypeFamilies_ to have a mechanism for implementing
type level functions.

{% highlight haskell %} 
type family Add (a :: Nat) (b :: Nat) :: Nat
type instance Add  Z    b = b
type instance Add (S a) b = S (Add a b)
{% endhighlight %}

Again we can try to inspect the kind of the type level function:
`:kind Add (S Z) (S Z)` which yields `Add (S Z) (S Z) :: Nat` as
expected.

To actually calculate the Fibonacci type we need Haskell to reduce the `Add`
expression. This is done using the `:kind! Add (S Z) (S Z)` operation.

{% highlight bash %} 
*FibType> :kind! Add (S Z) (S Z)
Add (S Z) (S Z) :: Nat
= 'S ('S 'Z)
{% endhighlight %}

We can now both define type level datatypes (kinds) and do operations on these.
Along this line I implemented Fibonacci straight forward in direct recursion.

{% highlight haskell %} 
type family Fibonacci (n :: Nat) :: Nat
type instance Fibonacci Z           = Z
type instance Fibonacci (S Z)       = (S Z)
type instance Fibonacci (S (S n))   = Add (Fibonacci n) (Fibonacci (S n))
{% endhighlight %}

To implement above type family I had to add the language extension
_UndecidableInstances_. This is because of the use of the `Add` type
family in the `Fibonacci` type family.

In this example, however, it is easy to see that the type family will
always converge. `Add` converges (and does not need
_UndecidableInstances_) and `Fibonacci` converges as the arguments
to the recursive applications are decreasing.

The last thing is to actually calculate Fibonacci. This can be done by
issuing the `:kind!` as earlier on and read the result. The returned
value is not easily readable but it is 13 as expected.

{% highlight bash %}
$ ghci FibType.hs -XDataKinds
...
*FibType> :kind! Fibonacci (S (S (S (S (S (S (S Z)))))))
Fibonacci (S (S (S (S (S (S (S Z))))))) :: Nat
= 'S ('S ('S ('S ('S ('S ('S ('S ('S ('S ('S ('S ('S 'Z))))))))))))
{% endhighlight %}

The code is as usual available on
[Github](https://github.com/madsbuch/fibonacci/tree/master/haskell).

Types alone does, however, not make a program. We need some kind of value
level implementation. In the next section I make a similar implementation
of the type level functions on the value level. Coupling the type and the
value in an one-to-one correspondence yields
[dependent programming](https://wiki.haskell.org/Dependent_type)

Dependent types are used to statically reason about programs and
put up guarantees that can be checked on compile time.
In this case we can set up the guarantee
that the code _actually_ calculates the _n_'th Fibonacci number.

## Implementing Dependent Typing
With dependent typing the type of a term _depends_ on its value. In
Haskell we can benefit from making some parts of the types dependent on
its values. This could for examble be statically sized lists to make
matrix operations typesafe.

In this post I implement complete dependency between terms and their type.
The type for the Fibonacci term was implemented above. Now we just need to
write some code that couples it to the value level.

First we couple the datatype. We implement a value level datatype,
`SNat`, which statically embed the size of the value.

{% highlight haskell %} 
data SNat (a :: Nat) where
    SZ   :: SNat Z
    SS   :: SNat n -> SNat (S n)
{% endhighlight %}

In the above the value level constructor has the type `SNat Z`, where we
informally can read that it is the _zero_ element.

The successor value constructor has the type `SNat n -> SNat (S n)`
where we construct the type according to the constructed value.

An important property of above datatype is the bijection between
a datatype and its type. This is what we use to statically reason about
our programs and make sure certain guarantees are held.

After that we implement the addition function

{% highlight haskell %} 
add :: SNat m -> SNat n -> SNat (Add m n)
add SZ      b = b
add (SS a)  b = SS (add a b) 
{% endhighlight %}

The value level add function stays completely true to the type-level
add function and is together with the Fibonacci function below fairly
self explanatory.

{% highlight haskell %} 
fibonacci :: SNat n -> SNat (Fibonacci n)
fibonacci SZ            = SZ
fibonacci (SS SZ)       = (SS SZ)
fibonacci (SS (SS n))   = add (fibonacci n) (fibonacci (SS n))
{% endhighlight %}

We have now build a Fibonacci function where the returned value is bijective
to its type. Hence we are sure that what is computed at runtime is something
we can predict on compile time.

# Applications for Dependent Programming
All above is perfectly good. But why bother writing so much more code
to just have a dependency between the value and its type? Well, for most
applications this is not necessary, but the technique can be used to set
strict guarantees in certain situations.

When you know the shape of the datatypes on compile-time it can be
advantageous to model this shape into the program. This could be
some uses of matrices.

When implementing for example neural networks, the upper bound on
the topology is usually known when writing the code. In this case
one could use type safe matrix libraries to implement this functionality.

# Conclusion
In this post I first implemented the Fibonacci function on the type level.
I made a type for the _n_'th element in the Fibonacci series. This was
done through data kinds and type families.

After this a value level implementation was implemented. It was implemented
in such a way that its type was bijective to its return value.

The dependency between the terms and the types was carried out through
GADTs which allows us to encode the type level natural in the types for
the values.

[^typeArith]: [wiki.haskell.org/Type_arithmetic](https://wiki.haskell.org/Type_arithmetic).