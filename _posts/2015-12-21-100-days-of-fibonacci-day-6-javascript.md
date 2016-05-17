---
layout: post
title: 100 Days of Fibonacci - Day 6, JavaScript
video: false
comments: false
---

This is an article in a series of articles. An overview of the entire
project can be found [here](/blog/100-days-of-fibonacci-overview/).

Until now I have focused on Fibonacci as a function of some input.
I have defined a function which returns a value when supplied an argument.
This yield a batch processing paradigm. However, the Fibonacci is defined as
a series, and as such it makes sense to treat it as a series.

In this blog post, I am going to treat Fibonacci as an unlimited stream of
numbers. To carry out this idea, I am going to use JavaScript. The reason
for this is that JavaScript is a well-known programming language and
because their event system is easy to understand. Later I will expand
the notion to other languages where the concept is known as streams,
signals, coinductive data structures, etc.

# Day 6 - JavaScript
The core JavaScript implementation is like the iterative implementation
I did [day 1 in C](/blog/100-days-of-fibonacci-day-1-c/).
It is expanded to directly emit an event carrying the result.

{% highlight javascript %}
// Start dispatching events
function startDispatcher(){
    var a=0;
    var b=1;
    var i=1;

    //We never stop the stream
    while(true){
        var tmp = a+b;
        b=a;
        a=tmp

        // emit
        emitter.emit('fib', {"fibn" : a, "n":i});
        i++;
    }
}
{% endhighlight %}

When this function is called it will emit a new event every time a
Fibonacci number has been calculated. But this does not work by itself. We
also need something to consume it. Following is a function that consume the
events and prints a `.` on the screen. When we hit _n = 10_ we write the
corresponding element in the Fibonacci series.

{% highlight javascript %}
// Listen on the event
emitter.on("fib" ,function(e){
    // Only write result when n is 10
    if(e.n == 10)
        process.stdout.write("\n" + e.fibn + "\n");
    else
        process.stdout.write(".")
});
{% endhighlight %}

When running the file through node we get following:

{% highlight javascript %}
$ node fib.js 
.........
55
...................................................................................................
{% endhighlight %}

As usual, the code is available
[here](https://github.com/madsbuch/snippets/blob/master/fibonacci/fib.js).

# JavaScript and Event Driven Programming
JavaScript was initially made as a scripting language for browsers and
has gained its popularity for programming user interfaces. With faster
processors and faster implementations of JavaScript the browser has got
more responsibility, and JavaScript has been turned into a general purpose
programming language. It has been ported to run on the server (with the
most widespread implementation as being "nodeJS") and databases. It
is now possible to develop large distributed applications in nothing but
JavaScript, which is also done.

So what about the Events? When programming user interfaces the interface
usually stands still until a button is pressed or an input field is
filled. Here the subsystem fires some events whenever something meaningful
is happening. It is then possible to add functions to be called on these
events.

This approach has been in use for a long time. But has certain disadvantages
when working on large systems. It quickly becomes messy when too many events
are to be handled and dispatched. But certain solutions already do exists.
Later on, we will take a look at Elm, Here we model the complete application
and let the system handle what part of the web page is updated. And don't
matter. It is fast.

# Conclusion
Í have introduced the notion of events which are going to support our
understanding of streams and signals for reactive programming later on.
This I have done in JavaScript which has turned into a general purpose
programming language primarily for full stack web development.