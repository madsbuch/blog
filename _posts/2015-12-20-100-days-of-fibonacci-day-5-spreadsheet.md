---
layout: post
title: 100 Days of Fibonacci - Day 5, Spreadsheet
video: false
comments: false
---
This is an article in a series of articles. An overview of the entire
project can be found [here](/blog/100-days-of-fibonacci-overview/).

Over the course of this project, small intermedios will occur.
Today is one of these. The past days I implemented Fibonacci
in what is considered "tradition" programming languages. They
have a formal syntax based on a grammar and an alphabet.

Today I am presenting Fibonacci through a tool that many people
use on a daily basis. Also, people who do not come out of a
computer science / engineering world.

# Day 5 - Spreadsheet
I simply implemented Fibonacci by hard coding two values, 0 and 1, in
two cells. These are the values we earlier on has seen as the base cases.
Thereafter I used the formula `=C6+C5` in the third cell. I could then
simply drag and drop that cell allowing the spreadsheet to calculate
the respective values.

![Spreadsheet Fibonacci](/blog/media/2015-12-20-100-days-of-fibonacci-day-5-spreadsheet/fib_ods.png)

The column left to the Fibonacci column is the index of the corresponding
Fibonacci number.

I have written that this method is dynamic programming using tabulation.
The reason for this is that I assume this being a pre-calculated table of
Fibonacci numbers. When one has this table she can quickly look up the number
she needs for further calculation. There might be other classifications
of this technique using other reasons.

As usual, the file is in my Fibonacci snippet folder and can be found
[here](https://github.com/madsbuch/snippets/blob/master/fibonacci/fib.ods).

# Spreadsheets as Programs
The main idea with this post is simply to try to broaden up the perception
of programming. Many people program on a daily basis when they do their
budgeting or quickly need to calculate combined costs of some product.

Programming should not be considered foreign, as it isn't. It is something
we all do. But more on this idea in a later post.

# Conclusion
Today I demonstrated a new programming language simply by drag-and-drop.
The idea was to broaden up the idea about what programming is. This is not
the last time I do so, even though I promise to present some more complex
concepts before that.