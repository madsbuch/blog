---
layout: post
title: 100 Days of Fibonacci - Day 7, Coq
video: false
comments: false
---

I have been of a couple of days over the Christmas and New Year.
But now it is time to start my [100 days of Fibonacci](http://buchi.dk/blog/100-days-of-fibonacci-overview/)
project again.

Today I chose Coq and I decided to look at proving properties about
programs. The key idea in this post is to prove that two different
implementations of the Fibonacci function indeed are equivalent.
Concretely I have chosen the direct recursive and the accumulated
recursive implementations as the subjects.

These implementations are the same
as in [the first article](/blog/100-days-of-fibonacci-day-0-haskell/).
This is done deliberately as they are idiomatic to functional programming.
Furthermore, they showcase quite well why this has a value. The directly
recursive implementation is easy to understand and stays close to both
the definition of Fibonacci and the common understanding of the function.
On the other hand, the recursive function with the accumulator is harder
to understand but provides a significant speedup.

In this article, I show that the rest of the program can be indifferent
to which implementation is in use. This is done buy proving theorems, which
is considerably more time consuming that providing tests. The effort returns
in favor the strongest guarantee we can have for the property, a mathematical
proof.

# Day 7 - Coq!
As mentioned the two implementations are the directly recursive and
the recursive with accumulation. These are both implementations we have
[seen before](http://buchi.dk/blog/100-days-of-fibonacci-day-0-haskell/).
But we use them again as they provide a good body for
showcasing proving program equivalence. Next we have the two implementations.

{% highlight coq %}
Fixpoint fib_direct (n : nat) : nat :=
  match n with
    | 0    => 0
    | S n' => match n' with
                | 0 => 1
                | S n'' => fib_direct n' + fib_direct n''
              end
  end.
{% endhighlight %}

Above is the directly recursive implementation and next is the
accumulated recursive implementation.

{% highlight coq %}
Fixpoint fib_accumulator (n : nat) (a : nat) (b : nat) : nat :=
  match n with
    | 0      => b
    | (S n') => fib_accumulator n' (a+b) a
  end.
(* An alias to scrape away unnecessary information *)
Definition fib_acc (n : nat) := fib_accumulator n 1 0.
{% endhighlight %}

The results can be calculated by the `Compute` command. It is
hereafter visible in the `*goals*` window (Assuming you use the
[proof assistant](https://coq.inria.fr/)).

{% highlight coq %}
Compute fib_direct 10.
Compute fib_acc 10.
{% endhighlight %}

A couple of things look a bit different. The function signature looks
kind of weird. Furthermore, we are doing some kind of match stuff with
something that looks a bit like numbers.

In the function signature, we use the `Fixpoint` to denote a recursive
function. This is because Coq encodes recursion in its type theory and
implement what's called iso-recursion.

For integers, Coq does not use the regular atomic data types we know from
ex. Java. Instead, it has (and only has) inductive data types. In these
[Peano naturals](https://en.wikipedia.org/wiki/Peano_axioms) has
been implemented. This means that zero is represented as `O`, one
as `S O`, two as `S (S O)` and so forth (this does not implement integers
bu only naturals, but for Fibonacci this is just fine).

Coq automatically expands syntactic decimal numbers to its
internal representation. Hence
5 is just syntactic sugar for `S S S S S O`.
This is the same representation I
[implemented in Prolog](http://buchi.dk/blog/100-days-of-fibonacci-day-4-prolog/) earlier on.

As usual, the code is available on [Github](https://github.com/madsbuch/snippets/blob/master/fibonacci/fib.v)).

It is necessary to be able to reduce data to atomic pieces to do the proving
we want to do. This is to create a total dependency between the type and its
value. Imagine Java's `int` type. This type holds, at least, $2^{32}$ different
values without the compiler knowing which. In that case, it is not possible to
reason about the return value of a function, which we need for the next
section.

# Coq - proof assistant
Coq is not meant as a general-purpose programming language. In fact, it is
a proof assistant. This means that it is not possible to interact with
the surrounding world. It is not possible to read command line arguments or
invoke system calls. On the other hand, Coq can prove properties.

In the above we had two different implementations of the Fibonacci
function. One might wonder whether these functions always return the same
value. That is, does `fib_direct 10` and `fib_acc 10` yield the same result?
This can easily be checked by running the code. The problem is when we
want to check for all values of `n`. Is it true that
`fib_direct n = fib_acc n`? This we can actually prove in Coq, and
so I have done.

The first step I took to prove equivalence of the implementations was to
characterize the Fibonacci function.

{% highlight coq %}
Definition specification_of_fibonacci (f : nat -> nat) :=
  f 0 = 0
  /\
  f 1 = 1
  /\
  forall n : nat,
    f (S (S n)) = f (S n) + f n.
{% endhighlight %}

This is, in fact, a proposition. Is is _true_ if we have a function that behaves
as the specification. This specification is very close to the directly 
recursive implementation.

The next thing is to prove that this definition is unambiguous. This means
that _f_ will behave identical independent to the actual implementation. In
Coq this looks like following (The whole code is available on
[Github](https://github.com/madsbuch/snippets/blob/master/fibonacci/fib.v)).

{% highlight coq %}
Lemma there_is_only_one_fib :
  forall f1 f2 : nat -> nat,
    specification_of_fibonacci f1 ->
    specification_of_fibonacci f2 ->
    forall n : nat,
      f1 n = f2 n.
Proof.
  unfold specification_of_fibonacci.
  intros f1 f2.
  intros [ bc_f1_0 [ bc_f1_1 ic_f1 ] ]. (* Destruct the conjunctive clauses *)
  intros [ bc_f2_0 [ bc_f2_1 ic_f2 ] ].
   
  (* strengthening the induction hypothesis *)
  assert (H_fib : forall m : nat,
                    f1 m = f2 m /\ f1 (S m) = f2 (S m)).
  intro m.
  ...
  apply hf.
Qed.
{% endhighlight %}

What happens above is that we specify a lemma. Given two functions, that
both satisfies the specification of the Fibonacci function, they yield the
same result for all values of _n_.

After the specification of the lemma, we provide an actual proof. this is a
series of commands that generates a function which in turn satisfies the _type_
of the lemma. This process is somewhat out of scope, and I will look into this
in another post.

After this, all we need to do is to prove that both the implementations indeed
satisfies the specification. Hereafter we can use the property that there is
only one Fibonacci function to prove the equivalence.

{% highlight coq %}
Lemma fib_direct_satisfies_specification :
  specification_of_fibonacci fib_direct.
Proof.
...
Qed.

Lemma fib_acc_satisfies_specification :
  specification_of_fibonacci fib_acc.
Proof.
...
Qed.
{% endhighlight %}

And the proof that they are equivalent

{% highlight coq %}
Theorem fib_direct_is_equivalent_to_fib_acc :
  forall n: nat,
    fib_acc n = fib_direct n.
Proof.
  intros n.
  rewrite (there_is_only_one_fib fib_acc 
                                 fib_direct 
                                 fib_acc_satisfies_specification
                                 fib_direct_satisfies_specification).
  reflexivity.
Qed.
{% endhighlight %}

The signature of `there_is_only_one_fib` was given above. We
specialize it with the arguments so we can rewrite our goal.
The goal is then rewritten with `forall n, fib_acc n = fib_direct n`.
This yields a goal which is trivially true. Namely
`fib_direct n = fib_direct n`.

We are now sure that for any _n_ we can provide to the directly recursive
implementation of Fibonacci, the other implementation yields the same
result.

# Conclusion
In this post, I implemented the Fibonacci function in the same way
as in the first article.
These are idiomatic to functional programming and as such it makes sense
to keep using them here.

They key concept introduced here is the notion of proving propositions.
I proved that the two implementations indeed behave identically. We have
scratched the surface when it comes to the underlying theory. But
already saw the fruits of the efforts: The direct implementation is slow
but easy to understand. The other implementation is faster but slightly more
complicated. We have used this to prove that they _always_ return the same
value for the same input.