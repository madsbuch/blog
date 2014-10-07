---
layout: post
title: "The 3 Pillars of Software Development"
quote: "Decomposing the testing process quickly shows that it consists of two components. First a specification is formulated, which is later verified."
video: false
comments: true
---
_Test driven development is for many programmers a well known concept.
Tests are written, and procedures are implemented trying to satisfy the tests
providing in a little green mark. When the green mark is not easily obtainable,
divide and conquer is applied with more and easier tests.
In this article I try to decompose the process of testing to that of specifying
and verifying. I relate this notion to places where verification are
mathematically used to bind a specification and an implementation._

## Introduction
In the software development community it has become obvious that software has to
be tested. The reasons are in plural: Keeping a maintainable codebase, assurance
of software correctness, up-to-date documentation, continuous
_something_[^cStar], etc.

For me, the two most important reasons to do software testing are the following:

1. Forced reasoning: To be able to express a specification, knowledge about the
   problem and the problem domain has to be understood.
2. Correctness assurance: Proper tests are an assurance that specifications are
   met. This, among other things, allows for continuous integration which is 
   crucial in projects needing to be flexible.

### Testing as verifying a specification
In the above I did a slight transition from using the term testing to use the
terms specifying and verifying. Decomposing the testing process quickly
shows that it consists of two components. First a specification is formulated,
which is later verified against the actual implementation.

This split is necessary to be able to talk about the process in a broader term.
Within proven software, specifications are mathematically proven to be
implemented in a correct manner, hence the term testing is to vague (at least in
my consciousness) as we do not necessarily test a finite set of the input for a
given procedure.

### The premises
Implicitly, and now explicitly, I advocate testing. My opinion is that testing
should take up as many resources as implementing. Of course this opinion is not
strict, and many condition are in play when deciding the amount of resources
spend on verifying code. I trust the reader in knowing when to test. At least
it is not the main aim for this article.

### Examples
I have drawn examples from two worlds: One from the engineering world
represented by examples in Java. The other one is Coq[^coq]. Coq is a theorem
prover performing formal proof verification. That means that we can be sure what
is expressed also holds.

## Three Pillars
The premises for the article is now set, and the three pillars can be
introduced. I will allow to start with specification. Intuitively it might seem
more obvious to start with implementation as it is the most commonly thought of
part of the programming process. But for being able to start implementing, a
specification of some form must be present, implicitly or explicitly.

After the specification the implementation is introduced and at last they are
bound using the verification.


### Specification
Specifications ranges from the informal human readable text string, used to
specify the name of a testfunction in a given testsuite, to a formal definition.
No matter by which means the specifications is expressed the process of
writing it is still very important. It forces reasoning about the application.

{% highlight java %}
import org.junit.*;
 
public class TestAddition {
  @Test
  public void testThatAddAdds(){
    //[...]
  }
}
{% endhighlight %}
The above is an example drawn from Java, where a method is named to specify what
we are going to test. The function name is here seen as the specification for
what we are going to verify. It is given that the specification is informally
defined.

{% highlight coq %}
Definition specification_of_addition (f: nat -> nat -> nat) :=
  (forall b : nat,
     f 0 b = b)
  /\
  (forall a b : nat,
     f (S a) b = f a (S b)).
{% endhighlight %}
This example of a specification is, on the other hand, thoroughly expressed in
a specific syntax.

It should be noted that there is a difference on the two examples. The first one
is addition in Java's type system, integers with a given precision. In Coq we
show that it holds for all natural numbers.

### Implementation
The implementation is, usually, the well known part. Following is a suggestion
for some code that implements their specification. So far, though, you only have
my word that what I have expressed is also what is implemented.

{% highlight java %}
public class Addition {
  public int add(int a, int b){
    return a + b;
  }
}
{% endhighlight %}
The Java example should come as no surprise.

{% highlight coq %}
Fixpoint add a b : nat :=
  match a with
    | 0    => b
    | S a' => add a' (S b)
  end.
{% endhighlight %}
The implementation in Coq is on the other hand not as straight forward. Two type
constructors are given: $$S : nat \rightarrow nat$$ and $$O : nat$$ (which have the
alias 0). Addition is here build on top of this.


### Verification
For the last part we have the verification. Here we bond the implementation to
the specification.

{% highlight java %}
public class TestAddition {
  @Test
  public void testThatAddAdds(){
    //Assignment
    Addition adder = new Addition();
		
    //Action
    int shouldBe5 = adder.add(2, 3),
        shouldBe7 = adder.add(-2, 9),
        shouldBe5 = adder.add(8, -3),
        shouldBeMinus7 = adder.add(-4, -3);
		
    //Assert
    assertEquals(shouldBe5, 5);
    assertEquals(shouldBe7, 7);
    assertEquals(shouldBe5, 5);
    assertEquals(shouldBeMinus7, -7);
  }
}
{% endhighlight %}
For the Java, you still have to accept that you only have my word for
correctness regarding the specification. As the specification is completely
informal, the assertions is a product of my interpretation of the specification
and programming skills.

{% highlight coq %}
Lemma add_satisfies_specification_of_addition :
  specification_of_addition add.
Proof.
  unfold specification_of_addition.
  split.
    intro b.
    unfold add.
    reflexivity.
    
    intros a b.
    induction a as [|a' IHa'].
      unfold add.
      reflexivity.
      
      unfold add.
      fold add.
      reflexivity.
Qed.
{% endhighlight %}

For Coq, on the other hand, the specification are read by the theorem prover.
Here I provide a _certificate_ that binds the specification to the
implementation. Coq afterwards verifies that this certificate is valid.

## Discussion
This is one way of relating the testing process to the process of building
formally verified software. Testing should be applied in both cases, as
specifications need to be verified. This is a tools for humans to verify
_semantic preservation_ when communicating their thoughts to a computer, hence
the KISS principle[^kiss] applies; It is after all simpler to invoke a function
with a given input and provide expected result, than formulating a formal
specification.

## Further Reading
My offset in this article was a basic approach to testing using Java and formal
verification using Coq. That said, all mainstream languages have their own means
of testing. For formal verification other tools exists. This is evident from the
existing project, e.g. the seL4 microkernel does not use Coq for the verification
part.

Resources for formal verification using Coq are plenty. Some good ones are
Software Foundations[^softFound] and Certified Programming with Dependent
Types[^cpdt].

There are also some interesting projects using the formal verification on very
real problems. The __CompCert__[^CompCert] certified compiler is a formally
proven compiler for C.

In a bigger perspective the __seL4__[^sel4] is a formally proven microkernel.

Why are these necessary? one might ask. We have the GNU compiler and Linux. They
are both stable and performs quite well? The thing is that some applications
can't afford that a bug is not found until running that very specific instance
of code.

## Conclusion
Even though it might seem kind of magic that we can formally verify a piece of
software, we still have to remember what we verify. We do not verify that the
software does what it is supposed to do. We verify that the software does
what we have specified in the specification.

By this I emphasise the importance of doing simple and easily understandable
testing, were one can easily verify that the output is correct. I encourage
people to play with their software and try to break it. After all they could
have expressed the specification wrong.

[^coq]: [Coq website](http://coq.inria.fr/)
[^softFound]: Benjamin C. Pierce et al., _[Software Foundations ](http://www.cis.upenn.edu/~bcpierce/sf/current/index.html)_
[^cpdt]: Adam Chlipala, _[Certified Programming with Dependent Types](http://adam.chlipala.net/cpdt/)_
[^CompCert]: [CompCert website](http://compcert.inria.fr/)
[^sel4]: [seL4 website](http://sel4.systems/)
[^kiss]: [en.wikipedia.org/wiki/KISS_principle](http://en.wikipedia.org/wiki/KISS_principle)
[^cStar]: Integration, delivery, deployment, etc.

