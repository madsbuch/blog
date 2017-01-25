---
layout: post
title: Proving Stuff in Haskell
author: Mads Buch
video: false
comments: false
---

This article is my give on the relationship between mathematical proofs and
programming languages. Many details on specific implementation have been
left out with the aim for clarity and conceptual coherency.

The source used in this article is available as
[a Gist](https://gist.github.com/madsbuch/12043c4ad1c1fd0a80008ffb443e29d7).

# Proofs and Programming
First, what is a proof? A proof is a series of deductive arguments, such that
the proposition is justified. This description might seem quite
abstract, so let us look at a concrete example using the Peano naturals for
representing natural numbers.

$$
    1+1 = 2
$$

As the proposition has no quantifiers, we can directly use the first
argument to reduce the expression. That is, the one given by the
usual definition of addition over Peano numbers.
We then have the following expression.

$$
    2 = 2
$$

We still have a mathematical object, a propositional claim.
We know from the Peano axioms, that
syntactical equivalence satisfies reflexivity, symmetry, and transitivity.
Henceforth the properties of equality are satisfied, and we may end our
deductive sequence.

Next, we want to translate above into programming.

## In Haskell
From the [Curry–Howard correspondence](https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence)
we know that propositions are types and proofs are programs. Alright, so we
need to make an expression that has above expression as its type,
and a program which inhibits this type.

First, we need to define our objects: Peano naturals and Equality. We implement
Peano naturals the usual way. I elaborate on this in a 
[previous post](/blog/100-days-of-fibonacci-day-9-haskell-types/).
Equality is defined as follows.

```haskell
data Refl a b where
  Refl :: Refl a a
```

Evidently, we the value `Refl` can only inhibit the type `Refl a b` 
if the types `a` and `b` are identical.
Concerning the Curry-Howard correspondence, the `Refl` value also
has the unit type - it is not possible to attach further data to
the constructor.

We have defined equality in terms of reflection. However, we need equality
also to satisfy symmetry and transitivity. In the source, we have the code
needed for that.

We now want to make the type[^prefix] for our proof, or, the equivalent to the 
proposition stated above:

```haskell
onePlusOneEqualsTwo :: Refl (Add (S Z) (S Z)) (S (S Z))
```

The `Add` in the type definition is a type family defined in the source. It is
defined as we would usually define addition over Peano naturals.

To prove it we need to make an inhibitor to that type. The program
is very simple
for this case as the Haskell compiler reduces the type level expression 
per semantics of type families.

```haskell
onePlusOneEqualsTwo = Refl
```

As it compiles[^compiler] it shows that Haskell is content with the proof.

# Quantifiers
We want to abstract our proofs. In proving terminology, we do this through
quantifiers.

To have a more graspable problem that includes only quantification,
without induction, we detour to boolean algebra. Here we can
try to formalize De Morgan's theorem:

$$
    \forall a, b \in : \lnot( a \land b ) = \lnot a \lor  \lnot b
$$

here _a_ and _b_ can only assume two values, _true_ and _false_.

```haskell
deMorgan :: SBool a -> SBool b -> Refl (Not (And a b)) (Or (Not a) (Not b))
deMorgan STrue STrue   = Refl -- The first case
deMorgan STrue SFalse  = Refl
deMorgan SFalse STrue  = Refl
deMorgan SFalse SFalse = Refl
```

We simply provide an inhibitor to the type based on
pattern matching. This is the same as proving by case analysis.

To understand what goes on we instantiate the type expression 
in each case. Afterward, the compiler reduces per the semantics
of the `Not`, `And`, and `Or` type families. In the first case we instantiate
the type expression such that.

```haskell
deMorgan :: SBool Tru -> SBool Tru
    -> Refl (Not (And Tru Tru)) (Or (Not Tru) (Not Tru))
```

This instantiation is from the value `STrue` which has the type `SBool Tru`.
The compiler then reduces the expression and derives that
`Refl :: Refl Fls Fls`. From the interpreter, we get that.

```
*Proof> :t deMorgan STrue STrue
deMorgan STrue STrue :: Refl 'Fls 'Fls
```

# Induction
Many interesting properties we want to reason about includes unbound data.
That is, the data we think about is inductively defined. We now go back to the
examples considering natural numbers as they are a good medium for
discussing inductively defined data.

`plus_id_r` is the property that adding zero to
_n_ on the right side is the identity of _n_. `plus_id_l` is when we add 0
on the left side.

$$
    n+0 = n \ \text{(plus_id_r)} \\
    0+n = n \ \text{(plus_id_l)}
$$

`plus_id_l` is given directly from our definition of addition. But `plus_id_r`
needs to be proven, and we can do this inductively using following code.

```haskell
plus_id_r :: forall n. SNat n -> Refl (Add n Z)  n
plus_id_r Zero = Refl
plus_id_r (Succ x) = gcastWith (plus_id_r x) Refl
```

The first case is the base case. We know that the value is `Zero` and
hence we can derive the type to `Z`. It is immediately visible that `Refl`
inhibits the type `Refl Z  Z`.

The next case is the induction case. Here we fold out the value such that if
`n = Succ x`, then `x = n-1` - we reduce this on our argument. We justify that 
`Refl` also is an inhabitant in this case by calling `plus_id_r` on the
reduced value.

# Discussion
It is indeed possible to prove stuff in Haskell. But it is not further
practical. The reason is in particular because the language
is not designed with
the constructions we need, such as dependent types. We simulate
them through GADTs.

The closest languages to Haskell that are suited for
this is languages such as Idris and Gallina (Coq). They have all the facilities
needed for incorporating proofs into one's code.

If one has a software
development background firmly grounded in OOP (Java, C#), it requires quite
some time to wrap one's head around the new way to understand types.

That we can do above is mostly of academic interest: How do make _sure_ that
certain compilers indeed do what they should do etc. But the techniques are
becoming steadily more accessible to all programmers.

New languages, like Idris, come with type constructions to formally
reason about our software.

# Final Remarks
First, thanks to [Aslan Askarov](http://askarov.net/) for providing
valuable feedback on this article. It has been incorporated to provide a
more coherent article.

This article was a precursor for a presentation at a
[spare time teaching](http://www.sparetimeteaching.dk/about.php) event
at Aarhus university. Thanks to them for letting me host the presentation.

[^prefix]: We solely use prefix notation to simplify the syntax.
[^compiler]: Well, we need to set some compiler flags to make sure that all cases are covered.