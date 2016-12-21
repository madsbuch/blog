---
title: The Probability Monad
author: Mads Buch
layout: post
video: false
comments: false
papers:
    - ramsey2012
categories: Probabilistic Programming
---

<!-- Why is this interesting? -->
What are the foundational structures of probabilities? How do we design
a language making it easy to model probabilistic problems? Oftentimes
the modeling happens directly in terms of vectors and matrices,
but there are better ways.

In this article we explore a structure, the monad, that spans probabilities.
It is based on the article _Stochastic Lambda Calculus and Monads of 
Probability Distributions_ by Ramsey and Pfeffer.

The source for this article is available as a
[gist on Github](https://gist.github.com/madsbuch/5a8a1fc9b70621dd93dd70058754b126).

# A Monad
A monad is a mathematical structure with wide applications in
functional programming. The reason is that it helps hiding away
some state and provides primitives for progressing the structure in
a linear fashion.

It is useful to know about monads. Not just for understanding the probability
monad, but because they are an important tool to architect applications using
functional techniques.

A monad is a well elaborated subject, and as such I am not elaborating on it
more than its algebraic structure. To get a proper overview on there exists
plenty resources.

For our practical use of the monad we need to implement following functions
in a way such that they satisfy the monadic laws.

```haskell
(>>=)  :: m a -> (a -> m b) -> m b -- pronounced bind
return :: a -> m a
```

These are what constitutes monads.

# Probabilities
When talking about events and the probability that they happen we often
talk in terms of distributions. In its most abstract form a distribution
is simply a total function, i.e. all inputs needs to be mapped to outputs.
This is also the view we take on it and the monad turns out to be a way of
packaging these programs to only allow certain operations on them.

We only discuss discrete probability distributions here. This
makes it considerable easier to reason about, but also poses some limitations
as we can only model continuous distributions to an arbitrary degree.

On the distributions we want to do certain queries. Some often used ones are
the `expectation`, the `support`, and the `sample` query. The `expectation`
and the `support` queries are much easier to think about when only
considering discrete distributions.

Lastly we also need an operation for building distributions. In this work
we take offset in a single function called `choose` which act as a construct
that binary chooses between two distribution with a certain probability.

To end choosing we use the return construction, which is th same as the Dirac
distribution. In some papers and languages dirac is used as the name for
return.


# Class Hierarchy
To get an overview of the implementation of the probability monad
it makes sense to look at the class hierarchy we are considering.

Following figure shows the monads we are implementing.

<svg height="330" width="100%"><defs><marker id="triangle" viewBox="0 0 14 14" refX="0" refY="5" markerUnits="strokeWidth" markerWidth="10" markerHeight="10" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z"></path></marker></defs><line x1="4" x2="12" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="4" y1="8" y2="16" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="8" x2="16" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="16" x2="24" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="24" x2="32" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="32" x2="40" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="40" x2="48" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="48" x2="56" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="56" x2="64" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="64" x2="72" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="72" x2="80" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="80" x2="88" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="88" x2="96" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="96" x2="104" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="104" x2="112" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="112" x2="120" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="120" x2="128" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="128" x2="136" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="136" x2="144" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="144" x2="152" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="152" x2="160" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="160" x2="168" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="168" x2="176" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="176" x2="184" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="184" x2="192" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="192" x2="200" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="200" x2="208" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="208" x2="216" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="216" x2="224" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="224" x2="232" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="232" x2="240" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="240" x2="248" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="248" x2="256" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="256" x2="264" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="264" x2="272" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="272" x2="280" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="280" x2="288" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="288" x2="296" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="296" x2="304" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="304" x2="312" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="312" x2="320" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="320" x2="328" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="328" x2="332" y1="8" y2="8" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="332" x2="332" y1="8" y2="16" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="4" y1="16" y2="32" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="332" x2="332" y1="16" y2="32" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="4" y1="32" y2="48" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><text x="150" y="44" style="font-size:14px;font-family:monospace">M</text><text x="158" y="44" style="font-size:14px;font-family:monospace">o</text><text x="166" y="44" style="font-size:14px;font-family:monospace">n</text><text x="174" y="44" style="font-size:14px;font-family:monospace">a</text><text x="182" y="44" style="font-size:14px;font-family:monospace">d</text><line x1="332" x2="332" y1="32" y2="48" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="4" y1="48" y2="64" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="332" x2="332" y1="48" y2="64" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="4" y1="64" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="12" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="8" x2="16" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="16" x2="24" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="24" x2="32" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="32" x2="40" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="40" x2="48" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="48" x2="56" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="56" x2="64" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="64" x2="72" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="72" x2="80" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="80" x2="88" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="88" x2="96" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="96" x2="104" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="104" x2="112" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="112" x2="120" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="120" x2="128" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="128" x2="136" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="136" x2="144" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="144" x2="152" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="152" x2="160" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="160" x2="168" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="168" x2="172" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="172" x2="180" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="172" x2="172" y1="72" y2="80" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="176" x2="184" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="184" x2="192" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="192" x2="200" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="200" x2="208" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="208" x2="216" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="216" x2="224" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="224" x2="232" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="232" x2="240" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="240" x2="248" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="248" x2="256" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="256" x2="264" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="264" x2="272" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="272" x2="280" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="280" x2="288" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="288" x2="296" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="296" x2="304" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="304" x2="312" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="312" x2="320" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="320" x2="328" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="332" x2="332" y1="64" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="328" x2="332" y1="72" y2="72" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="172" x2="172" y1="80" y2="96" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="172" x2="172" y1="96" y2="112" style="stroke: rgb(0,0,0);stroke-width:1" marker-end="url(#triangle)"></line><line x1="4" x2="12" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="4" y1="120" y2="128" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="8" x2="16" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="16" x2="24" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="24" x2="32" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="32" x2="40" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="40" x2="48" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="48" x2="56" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="56" x2="64" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="64" x2="72" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="72" x2="80" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="80" x2="88" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="88" x2="96" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="96" x2="104" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="104" x2="112" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="112" x2="120" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="120" x2="128" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="128" x2="136" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="136" x2="144" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="144" x2="152" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="152" x2="160" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="160" x2="168" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="168" x2="176" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="176" x2="184" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="184" x2="192" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="192" x2="200" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="200" x2="208" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="208" x2="216" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="216" x2="224" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="224" x2="232" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="232" x2="240" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="240" x2="248" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="248" x2="256" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="256" x2="264" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="264" x2="272" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="272" x2="280" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="280" x2="288" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="288" x2="296" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="296" x2="304" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="304" x2="312" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="312" x2="320" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="320" x2="328" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="328" x2="336" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="336" x2="340" y1="120" y2="120" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="340" x2="340" y1="120" y2="128" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="4" y1="128" y2="144" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="340" x2="340" y1="128" y2="144" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="4" y1="144" y2="160" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><text x="94" y="156" style="font-size:14px;font-family:monospace">P</text><text x="102" y="156" style="font-size:14px;font-family:monospace">r</text><text x="110" y="156" style="font-size:14px;font-family:monospace">o</text><text x="118" y="156" style="font-size:14px;font-family:monospace">b</text><text x="126" y="156" style="font-size:14px;font-family:monospace">a</text><text x="134" y="156" style="font-size:14px;font-family:monospace">b</text><text x="142" y="156" style="font-size:14px;font-family:monospace">i</text><text x="150" y="156" style="font-size:14px;font-family:monospace">l</text><text x="158" y="156" style="font-size:14px;font-family:monospace">i</text><text x="166" y="156" style="font-size:14px;font-family:monospace">t</text><text x="174" y="156" style="font-size:14px;font-family:monospace">y</text><text x="190" y="156" style="font-size:14px;font-family:monospace">M</text><text x="198" y="156" style="font-size:14px;font-family:monospace">o</text><text x="206" y="156" style="font-size:14px;font-family:monospace">n</text><text x="214" y="156" style="font-size:14px;font-family:monospace">a</text><text x="222" y="156" style="font-size:14px;font-family:monospace">d</text><line x1="340" x2="340" y1="144" y2="160" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="4" y1="160" y2="176" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="340" x2="340" y1="160" y2="176" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="4" y1="176" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="12" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="8" x2="16" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="16" x2="24" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="24" x2="32" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="32" x2="40" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="40" x2="48" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="48" x2="56" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="56" x2="60" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="60" x2="68" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="60" x2="60" y1="184" y2="192" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="64" x2="72" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="72" x2="80" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="80" x2="88" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="88" x2="96" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="96" x2="104" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="104" x2="112" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="112" x2="120" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="120" x2="128" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="128" x2="136" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="136" x2="144" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="144" x2="152" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="152" x2="160" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="160" x2="168" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="168" x2="176" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="176" x2="180" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="180" x2="188" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="180" x2="180" y1="184" y2="192" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="184" x2="192" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="192" x2="200" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="200" x2="208" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="208" x2="216" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="216" x2="224" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="224" x2="232" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="232" x2="240" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="240" x2="248" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="248" x2="256" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="256" x2="264" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="264" x2="272" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="272" x2="280" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="280" x2="288" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="288" x2="296" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="296" x2="300" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="300" x2="308" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="300" x2="300" y1="184" y2="192" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="304" x2="312" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="312" x2="320" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="320" x2="328" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="328" x2="336" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="340" x2="340" y1="176" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="336" x2="340" y1="184" y2="184" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="60" x2="60" y1="192" y2="208" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="180" x2="180" y1="192" y2="208" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="300" x2="300" y1="192" y2="208" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="60" x2="60" y1="208" y2="224" style="stroke: rgb(0,0,0);stroke-width:1" marker-end="url(#triangle)"></line><line x1="180" x2="180" y1="208" y2="224" style="stroke: rgb(0,0,0);stroke-width:1" marker-end="url(#triangle)"></line><line x1="300" x2="300" y1="208" y2="224" style="stroke: rgb(0,0,0);stroke-width:1" marker-end="url(#triangle)"></line><line x1="4" x2="12" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="4" y1="232" y2="240" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="8" x2="16" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="16" x2="24" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="24" x2="32" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="32" x2="40" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="40" x2="48" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="48" x2="56" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="56" x2="64" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="64" x2="72" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="72" x2="80" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="80" x2="88" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="88" x2="96" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="96" x2="104" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="104" x2="112" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="112" x2="116" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="116" x2="116" y1="232" y2="240" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="140" x2="148" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="140" x2="140" y1="232" y2="240" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="144" x2="152" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="152" x2="160" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="160" x2="168" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="168" x2="176" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="176" x2="184" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="184" x2="192" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="192" x2="200" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="200" x2="208" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="208" x2="216" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="216" x2="220" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="220" x2="220" y1="232" y2="240" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="244" x2="252" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="244" x2="244" y1="232" y2="240" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="248" x2="256" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="256" x2="264" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="264" x2="272" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="272" x2="280" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="280" x2="288" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="288" x2="296" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="296" x2="304" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="304" x2="312" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="312" x2="320" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="320" x2="328" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="328" x2="336" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="336" x2="340" y1="232" y2="232" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="340" x2="340" y1="232" y2="240" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="4" y1="240" y2="256" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="116" x2="116" y1="240" y2="256" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="140" x2="140" y1="240" y2="256" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="220" x2="220" y1="240" y2="256" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="244" x2="244" y1="240" y2="256" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="340" x2="340" y1="240" y2="256" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="4" y1="256" y2="272" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><text x="14" y="268" style="font-size:14px;font-family:monospace">E</text><text x="22" y="268" style="font-size:14px;font-family:monospace">x</text><text x="30" y="268" style="font-size:14px;font-family:monospace">p</text><text x="38" y="268" style="font-size:14px;font-family:monospace">e</text><text x="46" y="268" style="font-size:14px;font-family:monospace">c</text><text x="54" y="268" style="font-size:14px;font-family:monospace">t</text><text x="62" y="268" style="font-size:14px;font-family:monospace">a</text><text x="70" y="268" style="font-size:14px;font-family:monospace">t</text><text x="78" y="268" style="font-size:14px;font-family:monospace">i</text><text x="86" y="268" style="font-size:14px;font-family:monospace">o</text><text x="94" y="268" style="font-size:14px;font-family:monospace">n</text><line x1="116" x2="116" y1="256" y2="272" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="140" x2="140" y1="256" y2="272" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><text x="150" y="268" style="font-size:14px;font-family:monospace">S</text><text x="158" y="268" style="font-size:14px;font-family:monospace">u</text><text x="166" y="268" style="font-size:14px;font-family:monospace">p</text><text x="174" y="268" style="font-size:14px;font-family:monospace">p</text><text x="182" y="268" style="font-size:14px;font-family:monospace">o</text><text x="190" y="268" style="font-size:14px;font-family:monospace">r</text><text x="198" y="268" style="font-size:14px;font-family:monospace">t</text><line x1="220" x2="220" y1="256" y2="272" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="244" x2="244" y1="256" y2="272" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><text x="262" y="268" style="font-size:14px;font-family:monospace">S</text><text x="270" y="268" style="font-size:14px;font-family:monospace">a</text><text x="278" y="268" style="font-size:14px;font-family:monospace">m</text><text x="286" y="268" style="font-size:14px;font-family:monospace">p</text><text x="294" y="268" style="font-size:14px;font-family:monospace">l</text><text x="302" y="268" style="font-size:14px;font-family:monospace">e</text><line x1="340" x2="340" y1="256" y2="272" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="4" y1="272" y2="288" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><text x="38" y="284" style="font-size:14px;font-family:monospace">M</text><text x="46" y="284" style="font-size:14px;font-family:monospace">o</text><text x="54" y="284" style="font-size:14px;font-family:monospace">n</text><text x="62" y="284" style="font-size:14px;font-family:monospace">a</text><text x="70" y="284" style="font-size:14px;font-family:monospace">d</text><line x1="116" x2="116" y1="272" y2="288" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="140" x2="140" y1="272" y2="288" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><text x="158" y="284" style="font-size:14px;font-family:monospace">M</text><text x="166" y="284" style="font-size:14px;font-family:monospace">o</text><text x="174" y="284" style="font-size:14px;font-family:monospace">n</text><text x="182" y="284" style="font-size:14px;font-family:monospace">a</text><text x="190" y="284" style="font-size:14px;font-family:monospace">d</text><line x1="220" x2="220" y1="272" y2="288" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="244" x2="244" y1="272" y2="288" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><text x="262" y="284" style="font-size:14px;font-family:monospace">M</text><text x="270" y="284" style="font-size:14px;font-family:monospace">o</text><text x="278" y="284" style="font-size:14px;font-family:monospace">n</text><text x="286" y="284" style="font-size:14px;font-family:monospace">a</text><text x="294" y="284" style="font-size:14px;font-family:monospace">d</text><line x1="340" x2="340" y1="272" y2="288" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="4" y1="288" y2="304" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="116" x2="116" y1="288" y2="304" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="140" x2="140" y1="288" y2="304" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="220" x2="220" y1="288" y2="304" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="244" x2="244" y1="288" y2="304" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="340" x2="340" y1="288" y2="304" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="4" y1="304" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="4" x2="12" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="8" x2="16" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="16" x2="24" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="24" x2="32" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="32" x2="40" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="40" x2="48" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="48" x2="56" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="56" x2="64" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="64" x2="72" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="72" x2="80" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="80" x2="88" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="88" x2="96" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="96" x2="104" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="104" x2="112" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="116" x2="116" y1="304" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="112" x2="116" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="140" x2="140" y1="304" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="140" x2="148" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="144" x2="152" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="152" x2="160" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="160" x2="168" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="168" x2="176" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="176" x2="184" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="184" x2="192" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="192" x2="200" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="200" x2="208" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="208" x2="216" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="220" x2="220" y1="304" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="216" x2="220" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="244" x2="244" y1="304" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="244" x2="252" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="248" x2="256" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="256" x2="264" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="264" x2="272" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="272" x2="280" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="280" x2="288" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="288" x2="296" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="296" x2="304" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="304" x2="312" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="312" x2="320" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="320" x2="328" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="328" x2="336" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="340" x2="340" y1="304" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line><line x1="336" x2="340" y1="312" y2="312" stroke="rgb(0,0,0)" stroke-width="1" stroke-linecap="round" stroke-linejoin="mitter"></line></svg>

The lowermost three boxes are the most interesting. Those are the queries, the
operations we can use on a distribution to make sense of it.

# The Expectation Monad
This section elaborates on a measure theoretic approach to the
probability monad. We represent the distribution as a continuation
that takes a measure function and returns an expectation. 

First we will take a look at our monad type.

```haskell
-- Probability Monad
newtype PExp a = P (( a -> Double) -> Double)
```

It is worth looking into what happens here. We have the constructor
`PExp`, that takes a function, a _measure function_, and gives the
expectation.

The monadic structure of probability distributions is in this setting
implemented as follows.

```haskell
instance Monad P where
  return x = P (\h -> h x)
  (P d) >>= k =
       P (\h -> let apply (P f) arg = f arg
                    g x = apply (k x) h
                  in d g)
```

From here the expectation monad is easily implemented as the whole
monad type is build around it.

```haskell
instance ExpMonad PExp where
  expectation h (PExp d) = d h
```

We could now start to do experiments base on this. But we would much
rather like to play with `support` and `sample` also. These, however,
are quite difficult to implement around the type of `PExp` so we are
going to attack this from another angle.

# Generalizing
The above implementation of the monad type is not very suited for other than
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
to used directly by the queries.

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