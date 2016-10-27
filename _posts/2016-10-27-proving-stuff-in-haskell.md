---
title: Proving Stuff in Haskell
author: Mads Buch
---

This is my give on the relation ship between mathematical proofs and
programming. Many details on exact implementation have been left out
with the aim of clarity and conceptual coherency.

The source for this example is available as
[a Gist](https://gist.github.com/madsbuch/12043c4ad1c1fd0a80008ffb443e29d7)

# Proofs and Programming
First, what is a proof? A proof is an inhibitor of a proposition. A proposition
is said to be true, if there exists any such inhibitor. This might seem quite
abstract, so let us look at a concrete example written in ordinary numbers. We
will from now on use it as syntactic sugar for Peano axioms.

$$
    1+1 = 2
$$

As the proposition has no quantifiers we can directly reduce the expression
using the reduction rules from the Peano axioms. without further ado we can
write above expression in the completely reduced form:

$$
    2 = 2
$$

Now we have a mathematical object, a propositional claim, for which we
can check if it has an inhibitor. We know from the Peano axioms that
syntactical equivalence satisfies reflexivity, symmetry, and transitivity.
Henceforth we have an inhibitor.

Next we want to translate above into programming.

## In Haskell
From the [Curry–Howard correspondence](https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence)
we know that propositions are types and proofs are programs. Alright, so we
need to make an expression that has above expression as its type,
and a program which inhibits this type.

First we need to define our objects: Peano naturals and Equality. Peano
natural are implemented the usual way. This is well elaborated in 
[previous posts](/blog/100-days-of-fibonacci-day-9-haskell-types/).
Equality is defined as follows.

```haskell
data Refl a b where
  Refl :: Refl a a
```

Evidently the only values that can inhibit this type have to have the same
types. With respect to the Curry Howard correspondence this is also a unit 
type - it is not possible to attach further data to the constructor.

We now want to make the type[^prefix] for our proof, or, the equivalent to the 
proposition stated above:

```haskell
onePlusOneEqualsTwo :: Refl (Add (S Z) (S Z)) (S (S Z))
```

To prove it we need to make an inhibitor to the that type. This is very simple
for this case as the type level expression can be directly reduced.

```haskell
onePlusOneEqualsTwo = Refl
```

As it compiles[^compiler] it shows that Haskell is content with the proof.

# Quantifiers
We want abstract our proofs. In proving terminology this is done through
quantifiers. To have a more graspable problem that includes only quantification
without more complicated concepts, we deroute to boolean algebra. Here we can
try to formalize the De Morgan theorem:

$$
    \forall a, b \in : \lnot( a \land b ) = \lnot a \lor  \lnot b
$$

here _a_ and _b_ can only assume two value, _true_ and _false_.

```haskell
deMorgan :: SBool a -> SBool b -> Refl (Not (And a b)) (Or (Not a) (Not b))
deMorgan STrue STrue   = Refl
deMorgan STrue SFalse  = Refl
deMorgan SFalse STrue  = Refl
deMorgan SFalse SFalse = Refl
```

It is evident that we simply provide an inhibitor to the type based on simple
pattern matching. This is the same as proving by case analysis.

To understand what goes on we can consider the type expression to be
instantiated in each case. Afterwards it is reduced per the semantics
of the `Not`, `And`, and `Or` type families. In the first case we instantiate
the type expression such that

```haskell
deMorgan :: SBool Tru -> SBool Tru
    -> Refl (Not (And Tru Tru)) (Or (Not Tru) (Not Tru))
```

This instantiation is from the constructed types from the value
`STrue :: SBool Tru`.

The compiler then reduces the expression and derives that
`Refl :: Refl Fls Fls`

# Induction
Many interesting properties we want to reason about includes unbound data.
That is, the data we think about is inductively defined. We now go back to the
examples considering natural numbers as they are a perfect medium for
discussing inductively defined data.

`plus_id_r` is the property that.

$$
    n+0 = n
$$

That is, that adding zero to
_n_ on the right side is the identity of _n_. `plus_id_l` is when we add 0
on the left side.

`plus_id_l` is given directly from our definition of addition. But `plus_id_r`
needs to be proven, and we can do this inductively using following code.

```haskell
plus_id_r :: forall n. SNat n -> Refl (Add n Z)  n
plus_id_r Zero = Refl
plus_id_r (Succ x) = gcastWith (plus_id_r x) Refl
```

The first case is the base case. We know that the value is `Zero` and
the type for that is derived to `Z`. It is immediately visible that `Refl`
inhibits the type `Refl Z  Z`.

The next case is the induction case. Here we fold out the value such that if
`n = Succ x`, then ` = n-1` - we reduce this on our argument. We justify that 
`Refl` also is an inhabitant in this case by calling `plus_id_r` on the
reduced value.

# Discussion
It is indeed possible to prove stuff in Haskell. If one has a software
development background firmly grounded in OOP (Java, C#), it requires quite
some time to wrap ones head around the new way to understand types.

That we can do above is mostly of academic interest: How do make _sure_ that
certain compilers indeed do what they should do etc. But the techniques are
becoming steadily more accessible to all programmers. New languages like
Rust brings in type constructions to 

[^prefix]: We solely use prefix notation to simplify the syntax.
[^compiler]: Well, we need to set some compiler flags to make sure that all cases are covered.