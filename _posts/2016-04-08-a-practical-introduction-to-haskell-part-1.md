---
title: A Practical Introduction to Haskell, Part 1
author: Mads Buch
date: 8 April 2016
layout: post
video: false
comments: false
---

<!--{ Prerequisites for following along | }-->
This post series provides an introduction to Haskell through a
practical example. It
assumes no prior use of the language or functional programming experience.
A basic understanding of programming, knowledge of using the terminal in Linux,
and knowing how to install the software is, however, expected.

<!--{ Introduction to the article | grave serious formal }-->
This tutorial will walk you through setting up the environment using
[Haskell Stack](http://haskellstack.org), initializing projects, writing
code, interactively run it, test it, and build / deploy it.

<!--{ Introduction to the mini project | }-->
As a body of this investigation we are going to develop a little
interpreter. We will be able to do addition, multiplication, subtraction, 
and division on floating point numbers. Interpreters and compilers 
often made in functional programming languages as the paradigm provides
good tools for working with the structure.

<!--{ Overview of the article | }-->
A lot of material is presented in this talk. The core elements are as
follows:

1. Motivation. Why even consider functional programming languages?
2. Setting up the environment
3. Functional constructions: Algebraic datatypes and functions
4. Functional way of thinking

In the next (still unpublished) articles we will cover the rest:

* Setting up testing
* Input/Output
* Build / Deploying

Everything will be brief, but luckily it is greatly documented.
So after reading this, if you want to go further, give it away
and search the Internet!

# Why Functional programming
"I already know Java and it provides a well paying job. Why bother
spending time on this?"

<!--{ Why other paradigms | }-->
It shall not be a secret: There exists many paradigms and even more
ways to solve problems (This is evident from my
[100 days of Fibonacci](http://buchi.dk/blog/100-days-of-fibonacci-overview/)
challenge). None of these methods are right or wrong, but some fit better
in given situations than others.

<!--{ Better programmer | }-->
My proposition is that people in general become better problem
solvers / programmers by having multiple perspectives on the problem.
This is a key assumption for this presentation and can indeed be opposed.
This is, however, out of scope.

<!--{ Why functional | }-->
Functional programming is one of the key paradigms. The key benefits of
functional programming are the inherent tools for securing code correctness
and very good tools for architecting applications.
This comes from the use of types. It is furthermore extremely flexible from
natural use of higher order functions. This allows to build powerful 
abstractions that are type safe and still intuitive to use.

# Why Haskell
<!--{ Introduction to Haskell | }-->
Haskell is a general purpose programming language based in pure
functional programming. By general purpose it is understood that no
specific domains of problems, or industries, are targeted rather than
others. This is evident from its use in everything from webframeworks
to embedded software development for micro processors and high
performance scientific computing.

<!--{ Flexibility of Haskell | }-->
Haskell has mechanism for making embedded domain specific languages
(EDSL) to target certain tasks. This adds to its flexibility as we
are able to modify the language _syntax_ to accommodate our needs. This
is useful if one wants to make a framework for a certain industry or
class of problems.

<!--{ Haskell's community support | }-->
The Haskell repositories are also well equipped. We are going to use
Stackage as we use Stack. Otherwise community packages are usually pulled
from Hackage.

<!--{ Developer Tool chain for Haskell | }-->
the Haskell ecosystem is being heavily developed at the moment.
Especially the tool chain for writing, testing, and deploying code.
In this post we take offset in Stack. It is a large integrated
tool which depends on the already wide spread cabal. But enough
talking. The first thing to get started is to set up the environment
and get a hold on the first couple of commands.

# The Environment
<!--{ Introduction of Haskell Stack | }-->
As mentioned we use Stack to manage our environment. Stack
automatically downloads and sets up the
right compiler, builds, tests, and makes documentation for the project.
It is also capable of deploying using docker and much more.

<!--{ Installation of Haskell Stack | }-->
Stack can be installed on all major platform. Instructions are available
on the
[website](http://docs.haskellstack.org/en/stable/README/#how-to-install).

## Setting up a Project
Having Stack setup we can know create our project. This is done by the
subcommand `new`.

{% highlight bash %}
$ stack new awesomepreter
Downloading template ...
...
{% endhighlight %}

Stack creates a new folder, `awesomepreter`, and populates it
with boilerplate code and folders.

<!--{ post setup instructions |  }-->
After the project was created we are going to make Stack download and setup
the compilers we need. This is done by `cd`-ing into the directory and run the sub command `setup`.

        $ cd awesomepreter
        awesomepreter$ stack setup

We can now run the interpreter, `stack ghci`, and test, `stack test`. These
are the only two commands we are using for now.

We furthermore have 3 folders in the newly created project root.

* `app`:  Contains all code related to the execution. This usually includes
  code for controlling program flow, reading arguments etc.
* `src`:  This folder contains all the business logic.
* `test`: All our tests are placed in this folder.


## Packages
<!--{ Introduction to packages |  }-->
We use packages in order to leverage on the work of other talented people.
Nobody want to reinvent the deep plate, so we use others inventions.
Furthermore package systems are also used to organize the code we write.
For this organizations might setup their own package management systems.

<!--{ What we use | }-->
For this project we only use a few packages in order to test our code.
We use _HUnit_ and some supporting packages. 

<!--{ How to add packages | }-->
The packages we need are added by including following in the
`awesomepreter.cabal` file. It has to be added to `build-depends` under the
`test-suite awesomepreter-test` section as it is where we use it.

        , HUnit
        , testpack
        , test-framework
        , test-framework-hunit

<!--{ How to apply the changes | }-->
For the changes to apply to our project we need to ask stack to solve 
dependencies. This is a rather easy process done with a single command.

        awesomepreter$ stack solver --update-config

From here the packages are available in our test part of the project. But
before we will start using these packages we will start implementing the
interpreter.

# Functional Constructs
<!--{ We start coding | }-->
Now the environment is set up for our purposes, and we will return to some
some programming discussing. For this section we will turn our attention
on the `src` folder, which contains the primary part of our program.

<!--{ Cleanup preparations | }-->
First thing: A bit of cleanup. In the file `src/Lib.hs` we remove the
string `(someFunc)` from the beginning so it says `module Lib where`.
This is because we don't want to consider the module system at present
time.

<!--{ Recap on the project we are implementing | }-->
We can now continue on with the project: We wanted to build an interpreter
that interprets expressions like `2 + 5` and `3 + 7 / 8`. To do this
we consider
how to model the problem, and how evaluate instances of that model to a
result. In functional programming we model such things in algebraic
datatypes (ADT) and implement the evaluation as functions on that datatype.

## Algebraic Datatypes
<!--{ How we interpret ADT in this project | }-->
In this project we see an ADT as a construction which takes an _operation_
(the constructor) and some associated data. For the above example, `2 + 5`,
the operation is addition and the data is 2 and 5. Hence we make an addition
constructor taking the two number: `Add 2 5`.

<!--{ Arbitrarily nesting | }-->
Above is not the whole story as we need to satisfy out types. We want to
be able to build nested expressions, something like `Add 2 (Add 2 3)`. But
as the number, 2 and 3, have the type `Int` and Add has some other type this
is not possible. To get around this we make a constructor which just hold
the number: `Lit 4`. This is also an inhabitant of the same type as `Add`.

<!--{ This is done through ADTs | }-->
Haskell has ADTs through the `data` construction. Such a thing consists
of a type deceleration and a number of constructors each with some
associated data.

<!--{ Haskell implementation | }-->
We implement this in the `src/Lib.hs` file. First we change the module
signature to `module Lib where` and then we append the following lines
of code to the file.

{% highlight haskell %}
data ALang =
    Lit Float
  | Add ALang ALang
  | Mul ALang ALang
  | Sub ALang ALang
  | Div ALang ALang
  deriving Show
{% endhighlight %}

Then open the interpreter by issuing `stack ghci` from the project root.

After a bit of compiling a prompt should be visible and ready to take input

{% highlight bash %}
awesomepreter$ stack ghci
...
Ok, modules loaded: Lib, Main.
*Main Lib> 
{% endhighlight %}

Here we can write `Lit 42` for which we will see the evaluated result
`Lit 42.0`. Haskell did automatic conversion to floats, hence the trailing
0.

The __deriving Show__ we added in the end of the declaration makes Haskell
figure out how to show instances of this data type itself.

## Functions
To perform the actual interpretation of the expression we need to implement
a function. We want a function that takes an expression as input, and emits
the result as output.

<!--{ Pattern matching | }-->
The first construction we hit when defining functions for algebraic
datatypes is __pattern matching__.

{% highlight haskell %}
awesomepret (Lit x)     = x
{% endhighlight %}

<!--{Why pattern matching | }-->
Pattern matching is used to two things: Unpacking datatypes and making
specialized functions for each case in the datatype, and providing a way
to unpack data from constructors. The above
case is the simplest case, where we simply remove the constructor
and return the contents.

<!--{ Recursion as iteration | }-->
In pure functional programming __recursion__ is how we provide iteration.
Where we are normally used to have a language construction to provide this
functionality, we here implement it on our value-level functions

The this case we have a concrete tree. Each node is an operation and
the leaves are literals. By recursion we aggregate this tree
into a single value by using the build-in
operations form Haskell.

{% highlight haskell %}
awesomepret (Add a b)   = (awesomepret a) + (awesomepret b)
{% endhighlight %}

Th make the expressions easier to use, we can use a __let .. in ..__ expression
to make intermediate values. This makes big expressions easier to read and
write.

{% highlight haskell %}
awesomepret (Mul a b)   = let
                            ae = awesomepret a
                            be = awesomepret b
                          in
                             ae * be
{% endhighlight %}

This naturally leads to the __where__-expression. For our purpose this
is also an organizational tool. It makes the code easier to read.

{% highlight haskell %}
awesomepret (Sub a b)   = ae - be
  where
    ae = awesomepret a
    be = awesomepret b
{% endhighlight %}

Often we need to do something different based on the value of a variable.
In this case we need to check if we divide by 0, and throw an exception if
we do so. In Haskell we can use __guards__ for this. This is strictly more elegant than in-lining an _if-then-else_ expression.

In our case we don't want to allow devision by 0:

{% highlight haskell %}
awesomepret (Div a b)
               | (awesomepret b) == 0    = error "Division by 0"
               | otherwise               = (awesomepret a) / (awesomepret b) 
{% endhighlight %}

We now have a working interpreter we can use from the Haskell REPL.

    awesomepreter$ stack ghci 
    ...
    *Main Lib> awesomepret (Add (Lit 5) (Lit 2))
    7.0


# The Functional Way of Thinking
We  now have a working piece of code, and we can try to dissect it to
figure out how we reasoned to get to this result.

The data we worked on was organized as a tree. In pure functional programming
everything are trees. Even the program we express is a tree. When executing
the program these trees are reduced as much as possible. This new tree, 
often just a singly value, is the result of the computation.

We also see that we have no language constructions. Everything can be put in
variables and passed around. This provides a high degree of flexibility. We
have tools for expressing powerful abstractions, but also to make completely
unreadable code.

## To Imperative Languages
Coming from an imperative language there are some main differences one should
have in mind.

There are __no sequential__ ordering of expressions. In pure functional
programming we solely have functions. When we are not able to reduce a
function expression any further, we have our result. And yes, the result
is a function. To get greater insight in this the
[Lambda Calculus](https://en.wikipedia.org/wiki/Lambda_calculus) is a
good starting point.

We also have a whole new vocabulary about functions:

* __Higher order function:__ Functions can be provided as arguments and
  returned from functions.
* __First class citizen:__ Functions are passed around in variables like
  ordinary values. 
* __Partial Functions:__ In the case of Haskell it means that a function does
  not pattern match on all cases of the datatype.

This vocabulary is also available to mainstream programming languages such
as Java, C#, etc. But they are central to functional programming.

The last difference to imperative programming is the lack of control
structures. That's right, we don't have any `while`-loops,
`if`-expression, etc. But fear not. We can express the same computation
in functional programming. For the iteration idioms we use recursion
and for the conditionals we use pattern matching.

## What I want vs. What to do
When doing imperative programming we have an explicit _state_. This state
is concrete variables. When performing computation we modify the state
in a linear fashion. We might concatenate a couple of strings and add
some numbers. In the end we return either a value, a pointer to the
data we made, or nothing. The last case assumes that we did some effect
full programming and that we are able to read the results from somewhere
else. This is not how we do when thinking functional programming.

In functional programming we think about what we have and what we want.
These objects are reasoned about as inhabitants of a type. A type is here 
understood as set of elements.

A function is a map from one type to another. A whole program is understood
as a map from some initial element of a given type, to another. This map,
or function, is in Haskell called `main`.

# Conclusion
We have enough to formulate simple expressions and evaluate them in the
interpreter. Next thing, for practical Haskell programs, is to set up
automated testing. This is to allow great flexibility in software development.
After that we also need to think about how users can access the
functionality we have build. So we look into deploying.

