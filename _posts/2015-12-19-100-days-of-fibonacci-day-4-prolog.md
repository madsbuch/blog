---
layout: post
title: 100 Days of Fibonacci - Day 4, Prolog
video: false
comments: false
---

__Updates:__

* 26 January 2016: Complete rewrite of the section about the Structural
  Implementation of Fibonacci.

In my [100 days of Fibonacci](/blog/100-days-of-fibonacci-overview/)
project I looked at functional and imperative language. There are however
many more programming paradigms today we are going to dive into
one of the most used
[logic programming languages](https://en.wikipedia.org/wiki/Logic_programming).

Prolog is heavily used in the field of artificial intelligence. This is due
to the natural way of formulating computations in terms of predicates.
This allows for concise expression of tree structures that certain data might
exhibit.

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
The `is`-operator does explicit evaluation of the right-hand side
before it check whether it equals the left-hand side.

## A Structural Implementation of Fibonacci

To remove the use of that operator I have implemented Peano natural
numbers and defined what addition is. This allows us to implement
Fibonacci in a purely structural manner.

Fibonacci is a very simple function that requires 3 constructions:
a data type (natural numbers), data aggregation (addition), and
a way to define the recursive dependencies. That last thing is done
through recursively defined predicates in Prolog.

{% highlight prolog %}
% Definiition of the Peano numbers
nat(0).
nat(s(_)).

% Readable nats
nat_to_int(0, 0).
nat_to_int(s(N), R) :-
    nat_to_int(N, R1),
    R is R1 + 1.

int_to_nat(0, 0).
int_to_nat(X, s(R)) :-
    X1 is X-1,
    int_to_nat(X1, R).
{% endhighlight %}

Above is the data type. The first three lines are somewhat unnecessary
as Prologs typing discipline is implicit.

The next two block display how we convert between the arithmetic integers
and the naturals. We use pattern matching an a recursively defined predicate.
This construction is quite idiomatic for this problem.

Net we have an implementation of addition. Again we use pattern
matching in two defined predicates. The first predicate is the
base case while the second is the induction case.

{% highlight prolog %}
% Definition of addition ( add(A, B, Result ))
add(0, B, B).
add(s(A), B, R) :- add(A, s(B), R).
{% endhighlight %}

To understand what is happening one can think of two stacks of plates
where we move all plates piecewise from the left stack to the right.
When the left stack is empty, we stop and let the right one be the
result.

The last part in this implementation is the definition of Fibonacci.
Note that we implicitly subtract the number by 2 when we match in
the induction case. This is necessary as we don't have any functionality
for subtracting. General subtraction is not necessary as we only need to
subtract by 2. In the body of the predicate we add by 1, to get the subtraction
of 1.

{% highlight prolog %}
fib_peano(0, 0).
fib_peano(s(0), s(0)).
fib_peano(s(s(N)), R) :-
    fib_peano(N, A),
    fib_peano(s(N), B),
    add(A, B, R).

% Wrapper functions dealing with all the conversions
fib_peano_wrapper_get_res(A, B) :-
    int_to_nat(A, X),
    fib_peano(X, Y),
    nat_to_int(Y, B).
fib_peano_wrapper_get_arg(A, B) :-
    int_to_nat(B, Y),
    fib_peano(X, Y),
    nat_to_int(X, A).
{% endhighlight %}

The two last blocks are for reading arithmetic arguments. They simply
convert a number to the Peano structure before it run the `fib_peano`
predicate. Lastly it converts back to arithmetic integers to make it
easier to read the output.

First we take a look at how it would look without conversion from integers.
It is apparent that it is rather cumbersome to practically work with numbers
in this format.

{% highlight bash %}
?- fib_peano(s(s(s(s(s(s(s(s(s(s(0)))))))))), R), nat_to_int(R, X).
R = s(s(s(s(s(s(s(s(s(s(...)))))))))),
X = 55 .
{% endhighlight %}

The last piece of code for this post demonstrates how the wrappers work.
the first example simply returns the 10th Fibonacci number.
The second example is more interesting. It lets us derive which
index the Fibonacci number 55 has.

{% highlight bash %}
?- fib_peano_wrapper_get_res(10, B).
B = 55 .

?- fib_peano_wrapper_get_arg(A, 55).
A = 10 .
{% endhighlight %}

The last piece of code also solves the puzzle from 
[yesterday](/blog/100-days-of-fibonacci-day-3-scala/)
where I asked what _n_ is for $$fib(n) = 55$$.

The time and space complexity is generally not good when using this
representation. It is usually only used when reasoning about programs, 
i.e. proving different properties about programs. The idea about
structuring problems in terms of nothing but predicates is very
usable, and is indeed one of the reasons why Prolog is widely used.

As usual the files are available on
[Github](https://github.com/madsbuch/fibonacci/tree/master/prolog).

# Programming Through Predicates
Logic programming has its core in predicates. We define logical
propositions and make the subsystem derive what we need. This works
very well when we work on structures like lists and trees.

The Fibonacci function, however, works on natural numbers which
we usually represent as integers for direct translation to
machine instructions.

As seen above it was indeed possible to implement naturals structurally.
On a more general note, it is actually possible to model all problems in
a structural manner. This is exploited in amongst other languages
[Coq](/blog/100-days-of-fibonacci-day-7-coq/).



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
