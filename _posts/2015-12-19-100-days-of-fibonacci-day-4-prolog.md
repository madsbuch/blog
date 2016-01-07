---
layout: post
title: 100 Days of Fibonacci - Day 4, Prolog
video: false
comments: false
---
This is an article in a series of articles. An overview of the entire
project can be found [here](/blog/100-days-of-fibonacci-overview/).

So far I have looked at functional and imperative languages. There are however
many more programming paradigms. Today's language is one in which
we do not initially exploit idiomatically. Prolog is heavily used in
artificial intelligence industry and research. This is due to
the flexibility of the language and the natural way of formulating
computations in terms of predicates. Prolog allows us to derive
the arguments of these predicates.

In this blog post, i show how to implement Fibonacci in the directly
recursive style. As we will see this gives problems when we want to
exploit some Prolog features.

# Day 4 - Prolog
In Prolog, I defined a predicate that holds both the argument and the
result. A predicate is something that can only be true or false.
Therefore, the notion of return value is explicit parameters to the
predicates in Prolog.

{% highlight prolog %}
% Implementation using direct recursion.
fib_direct(0, 0).
fib_direct(1, 1).
fib_direct(N, R) :-
    X1 is N-1,
    X2 is N-2,
    fib_direct(X1, A),
    fib_direct(X2, B),
    R is A + B.
{% endhighlight %}

In above the base cases are defined as simple predicates. The first argument
is the nth Fibonacci number and the second is the result.

By writing `fib_direct(0, 0).` we denote that the predicate predicate
`fib_direct` is true when applied 0 for both arguments. In `swipl`
(SWI Prolog interpreter) we can write `fib_direct(0, X).` to make
it derive the second argument given the first argument.

{% highlight bash %}
$ swipl fib.pl 
% fib.pl compiled 0.00 sec, 4 clauses
Welcome to SWI-Prolog ...

For help, use ?- help(Topic). or ?- apropos(Word).

?- fib_direct(0, R).
R = 0 .
{% endhighlight %}

The third definition of the `fib_direct` predicate is the one that
allows us to calculate Fibonacci for arbitrary numbers. We can again
make prolog derive the result by supplying N

{% highlight bash %}
...
?- fib_direct(10, R).
R = 55 .
{% endhighlight %}

A problem here is that we use the `is/2` notation. It enforces evaluation
of the right side and puts the result in the variable on the left side.
This is something we want to avoid in logical programming, as we need the
right side to be defined. The reason is that we can not choose what to derive anymore.

Yesterday I put up a little challenge to try to figure out where 55 came from.
This challenge is ideal to solve in Prolog as we can use the
same code to derive what _n_ is for a given Fibonacci number. We would have
done something like.

{% highlight bash %}
?- fib_direct(N, 55).
ERROR: fib_direct/2: Arguments are not sufficiently instantiated
{% endhighlight %}

But as you can see it fails. This is because of the use of `is/2`.

To remove the use of that operator I have implemented Peano natural
numbers and defined what addition is. This allows us to implement
Fibonacci in a purely structural manner.

{% highlight prolog %}
% Definiition of the Peano numbers
nat(0).
nat(s(_)).

% Readable nats
read_nat(0, 0).
read_nat(s(N), R) :-
    read_nat(N, R1),
    R is R1 + 1.

% Definition of addition ( add(A, B, Result ))
add(0, B, B).
add(s(A), B, R) :- add(A, s(B), R).
    
fib_peano(0, 0).
fib_peano(s(0), s(0)).
fib_peano(s(s(N)), R) :-
    fib_peano(N, A),
    fib_peano(s(N), B),
    add(A, B, R).
{% endhighlight %}

And we don't need to use explicit evaluation.

{% highlight bash %}
?- fib_peano(s(s(s(s(s(s(s(s(s(s(0)))))))))), R), read_nat(R, X).
R = s(s(s(s(s(s(s(s(s(s(...)))))))))),
X = 55 .
{% endhighlight %}

Note the `read_nat`. This is simply to make it readable.

We can put a number in on the result side, and make Prolog
derive from where is came (I am not using 55, but a smaller number, 5)

{% highlight bash %}
?- fib_peano(N, s(s(s(s(s(0)))))), read_nat(N, X).
N = s(s(s(s(s(0))))),
X = 5 .
{% endhighlight %}

And the result is indeed correct. If we provide a number that
either is not in the Fibonacci series or a large number (55 was too
large for me) Prolog will run for a long time and eventually
overflow the stack.

The time and space complexity is generally not good when using this
representation. It is usually only used when reasoning about programs
(i.e. proving different properties about programs). This makes these
example practically unusable.

The file can be downloaded
[here](https://github.com/madsbuch/snippets/blob/master/fibonacci/fib.pl).

# Programming Through Predicates
Logic programming has its core in predicates. We define logical
propositions and make the subsystem derive what we need. This works
very well when we work on structures like lists and trees. The Fibonacci
function, however, works on natural numbers which we usually represent as
integers for direct translation to machine instructions.
However, as I showed, naturals are indeed structural when using the Peano
axioms.

Actually, all problems may be expressed in structures. This we will
get back to when discussing theorem provers.

# Conclusion
Today I looked at Prolog. I implemented the Fibonacci function
in a logic context. We saw that We didn't exploit Prologs ability
to derive the argument to Fibonacci. To circumvent that, I
built a structural implementation of natural, using Peano,
so we didn't have to make an explicit evaluation. This implementation
had the ability to derive what number a given Fibonacci number is,
even though it is not practical.
