---
title: The Probability Monad
author: Mads Buch
layout: post
video: false
comments: false
papers:
    - ramsey2012
---

<!-- Why is this interesting? -->
What are the foundational structures of probabilities? How do we design
a language making it easy to model probabilistic problems? Oftentimes
nowadays the modeling happens directly in terms of vectors and matrices,
but there are better ways.

In this article we explore a structure, the monad, that spans probabilities.
It is based on the article _Stochastic Lambda Calculus and Monads of 
Probability Distributions_ by Ramsey and Pfeffer.

The source for this article is available as a
[gist on Github](https://gist.github.com/madsbuch/5a8a1fc9b70621dd93dd70058754b126).

# A Monad
A monad is a mathematical structure with wide applications in
functional languages. The reason is that it helps hiding away
some state, or a computational context, and exposes a more imperative
looking style.

It is useful to know about monads. Not just for understanding this article,
but also for the numerous other structures that span monads.

A monad is a well elaborated subject, and as such I am not elaborating on it
more than its algebraic structure. To get a proper overview on there exists
plenty of resources online.

For our practical use of the monad we need to implement following functions
in a way such that they satisfy the monadic laws.

```haskell
bind   :: m a -> (a -> m b) -> m b
return :: a -> m a
```

These are what constitutes monads.

# Probabilities
The probability of something happening is constituted by a distributions. It
describes potential outcomes that event might have and by what chance it
happens.

We talk only about discrete probability distributions in this article. This
makes it considerable easier to reason about, but also poses some limitations.

First, the limitations: We can't model continuous distributions. We can,
however, approximate them to an arbitrary degree.

On the distributions we want to do a certain number of queries. Here we
discuss `expectation`, `support`, and `sample`. The `expectation` and the
`support` queries are much easier to think about when only considering
discrete distributions.

Lastly we also need an operation for building distributions. In this work
we take offset in a single function called `choose`.

# Class Hierarchy
To get an overview of the implementation of the probability monad
it makes sense to look at the class hierarchy we are considering.

Following figure shows the monads we are implementing.

    +----------------------------------------+
    |                                        |
    |                  Monad                 |
    |                                        |
    +--------------------+-------------------+
                         |
                         V
    +-----------------------------------------+
    |                                         |
    |           Probability Monad             |
    |                                         |
    +------+--------------+--------------+----+
           |              |              |
           V              V              V
    +-------------+  +---------+  +-----------+
    |             |  |         |  |           |
    | Expectation |  | Support |  |  Sample   |
    |    Monad    |  |  Monad  |  |  Monad    |
    |             |  |         |  |           |
    +-------------+  +---------+  +-----------+

The lowermost three boxes are the most interesting. Those are the queries, the
operations we can use on a distribution to make sense of it.

# The Expectation Monad
First we will take a look at our monad type.

```haskell
-- Probability Monad
newtype P a = P (( a -> Double) -> Double)
```

It is worth looking a bit into what happens here. We have the constructor `P`,
that takes a function from a _measure function_ to a double. intuitively we
take a function that can calculate the expectation of some measure happening.

The monadic structure of probability distributions can be implemented as
follows:

```haskell
instance Monad P where
  return x = P (\h -> h x)
  (P d) >>= k =
       P (\h -> let apply (P f) arg = f arg
                    g x = apply (k x) h
                  in d g)
```

The return function is directly the same as the dirac distribution. That is,
it takes a single element of it's domain and spans a distribution where that
element is the only one.

# Generalizing
The above implementation of the monadic type is not very suited for other than
the expectation query. To make something more suited, we will try to stay true
to the paper and implement a type such that we can hold data as it is defined

```haskell
data P a where
    R :: a -> P a
    B :: P a -> (a -> P b) -> P b
    C :: Probability -> P a -> P a -> P a
```

The quick reader will see that we now use GADTs instead or ordinary ADTs. This
is due to the _Bind_ constructor. This poses a change of the type, and can not
be implemented by ADTs.

In the instances for the monad and probability monad we simply defer everything
to used directly by the queries

```haskell
-- P is a monad
instance Monad P where
  return x = R x
  d >>= k  = B d k

-- P is a probability monad
instance ProbabilityMonad P where
  choose p d1 d2 = C p d1 d2
```

This allows for definitions of the queries to be directly copied from the
paper. In addition to the `expectation` query we had before, we can now also
do `support` and `sampling`.

```haskell
instance SupportMonad P where
  support (R x) = [x]
  support (B d k) = concat [support (k x) | x <- support d]
  support (C p d1 d2) = support d1 ++ support d2
...
```

For the full source I once again refer to the
[gist on Github](https://gist.github.com/madsbuch/5a8a1fc9b70621dd93dd70058754b126)..

# Making Distributions
In this section we will take a look on how to construct some interesting
distributions and how the queries work on them.

As often when working with probabilities we will look on dices. To make
it easy we make a data type deriving `Enum` for easy enumeration.

```haskell
data Dice = One | Two | Three | Four | Five | Six
  deriving (Enum, Eq, Show, Read, Ord)
```

We can from here make a uniform distribution over the sides of a dice
like following.

```haskell
dist :: P Dice
dist = uniform [One .. Six]
```

The `uniform` method is a simple function that makes a uniform distribution
over a list using the `choose` function and the length of the list.

A support query can be performed as follows.

```haskell
example01a =
    let dist :: P Dice
        dist = uniform [One .. Six]
    in support dist
```

Yielding the result of `[One,Two,Three,Four,Five,Six]`. To create a posterior
distribution from a prior, we use the bind. This translates into elegant and
easily readable dependent distributions.

```
example02a =
    let dist :: P Dice
        dist = do
          d <- uniform [One .. Six]
          return (if d == Six then One else d)
    in support dist
```

This yields the result `[One,Two,Three,Four,Five,One]` Note how `One` appears
twice. This make immediate sense from the definition of the distribution
though we would have expected duplications removed. However, this is quite
easily done in Haskell.

The last example in the gist is one of the more interesting. The distribution
defined is potentially infinite. Intuitively it counts up with a probability
of a half and continues doing that.

```haskell
walk x =
   do bit <- uniform [True, False]
      if bit then
          return x
      else walk (x + 1)
```

So what happens when we throw the various queries on it?

The __support__ query should return a list of all potential outcomes of the
distribution, and so it does. running `example03a` returns
`[0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,^CInterrupted.`. If not
stopped it would continue counting up.

An important remark here is that
order of the `True` and `False` in the inner distribution. If we switch it
such that the inner distribution reads `uniform [False, True]` it will never
return anything. Intuitively one can understand it as the last element is
unfold first. In the current ordering the walk function creates a list which
is reducible though infinite.

Secondly we  have the __expectation__ query. issuing this yields an infinite
computation without any result. This is because we attempt to unroll the full
structure to get the expectation of an event.

Lastly we have the __sample__ query. This query works completely as expected yielding a list of samples. Issuing `example03c` will return
`[0,1,3,0,0,2,0,1,0,1]` which intuitively makes sense.

Though the expectation query is not possible through the current implementation
we could actually still get an idea about the expectations using the Monte
Carlo Technique. Sampling 10.000 elements of the walk distribution and taking
the frequency gives us following.

```haskell
> mc
[(0,4937),
 (1,2483),
 (2,1294),
 (3,630),
 (4,326),
 (5,159),
 (6,87),
 (7,34),
 (8,27),
 (9,10),
 (10,5),
 (11,5),
 (12,2),
 (15,1)]
```

Which is what we would expect.

# Concluding Remarks
We have discussed Ramsey and Pfeffer's paper in this article. It provides a
fundamental understanding based in programming languages on distributions and
probabilities. It is interesting because it makes languages like Hakary much
easier to understand. Furthermore it also greatly simplifies the hole world of
probabilities.  