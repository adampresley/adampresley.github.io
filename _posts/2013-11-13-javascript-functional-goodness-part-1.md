---
layout: post
title: JavaScript Functional Goodness Part 1
date: 2013-11-13 05:36:00
author: Adam Presley
status: Published
tags: development javascript
slug: javascript-functional-goodness-part-1
---

Slowly I have been trying to learn and work in more functional-style
programming constructs into my toolbelt. I'm not doing this for
any particularly nefarious reason other than I like to continue
to broaden my horizons as often as I can, and I find functional
programming concepts difficult, so that seems like a logical
place to start!

The starting point for me was chapter 6 of the online book
*Eloquent JavaScript* (<http://eloquentjavascript.net/chapter6.html>).
In chapter 6 the author provides a gentle introduction to
functional programming ideas and, starting from a few simple
concepts, builds up to some fairly sophisticated examples. So
I have taken these tutorials and put them into my own personal
JS library for reuse (<https://github.com/adampresley/rajo>). This
was step 1 in helping me get some of the material.

More recently I have been able to put some of this to good use.
In one project I'm working on I have a line chart with data fed to
it that looks like the following JSON.

    :::javascript
    {
       "seriesTotals": [
          3,
          77,
          54,
          65,
          64,
          56,
          41,
          14,
          63,
          99,
          63,
          30,
          41,
          24,
          12,
          42,
          53,
          51,
          50,
          35,
          27,
          22,
          58,
          65,
          40,
          51,
          44,
          27,
          12,
          66,
          64
       ],
       "labels":    [
          "Oct 13",
          "Oct 14",
          "Oct 15",
          "Oct 16",
          "Oct 17",
          "Oct 18",
          "Oct 19",
          "Oct 20",
          "Oct 21",
          "Oct 22",
          "Oct 23",
          "Oct 24",
          "Oct 25",
          "Oct 26",
          "Oct 27",
          "Oct 28",
          "Oct 29",
          "Oct 30",
          "Oct 31",
          "Nov 1",
          "Nov 2",
          "Nov 3",
          "Nov 4",
          "Nov 5",
          "Nov 6",
          "Nov 7",
          "Nov 8",
          "Nov 9",
          "Nov 10",
          "Nov 11",
          "Nov 12"
       ],
       "dayNames":    [
          "Sunday",
          "Monday",
          "Tuesday",
          "Wednesday",
          "Thursday",
          "Friday",
          "Saturday",
          "Sunday",
          "Monday",
          "Tuesday",
          "Wednesday",
          "Thursday",
          "Friday",
          "Saturday",
          "Sunday",
          "Monday",
          "Tuesday",
          "Wednesday",
          "Thursday",
          "Friday",
          "Saturday",
          "Sunday",
          "Monday",
          "Tuesday",
          "Wednesday",
          "Thursday",
          "Friday",
          "Saturday",
          "Sunday",
          "Monday",
          "Tuesday"
       ]
    }

The data is pretty simple. There is a top-level object that has three
keys, each one an array of data. The first array, **seriesTotals**
is an array of numeric totals. The second array is a set of dates, while
the third is an array of weekday names. Each item in each array lines up in order of
appearance. This data is then fed to a jqPlot line chart for visual
goodness.

The next order of business I had to address was to sum up the total
of the values so I could display it elsewhere on the page. One way to approach
this is using a simple variable and a loop, adding each item to the total sum.

    :::javascript
    var total = 0;

    for (var index = 0; index < data.seriesTotals.length; index++) {
        total += data.seriesTotals[index];
    }

    // total == 1413

This works fine and there is absolutely nothing wrong with it. But I am
wanting to try new ways to do this kind of thing, so I decided to go
functional with it. Warning, I am a complete n00b at this, and I may be
going about it the wrong way entirely. Hit me up on Google+ if you have comments
or questions about this. Here is a version using the *RAJO* utilities
I mentioned above.

    :::javascript
    require(["rajo.util"], function(Util) {
        var total = Util.reduce(
            0,
            data.seriesTotals,
            function(a, b) { return window.parseInt(a) + window.parseInt(b); }
        );

        // total == 1413
    });

I've added some extra line breaks for readability, but the original version is
a single line. Here's the breakdown.

In the first line I am using **Require.js** to declare my dependecies, and in this
case I am depending on the RAJO utility object. That will get injected in the
anonymous function as the variable *Util*.

From here I go on to declare a variable named *total* that will be assigned the result
of **Util.reduce()**. The idea behind a reduce function is that it will return a single
flattened value from a set (array) of values and a starting value. So in this case the
starting value is zero (0) and the array of values is found in our JSON data structure
**data.seriesTotals**.

Then comes the reduction function. It takes two arguments. The first is a previous value, while
the second is the current item from the items array. Note that on the first run the "previous"
value is the starting value. This function takes the two numbers, sums them up and returns
the result. Do that for each item in the array, and at the end you will have the total
value of the items in the array.

Is this better than the loop version? Aren't we technically looping anyway? Yes we are.
It seems one of the ideas behind functional programming is decomposing actions and
computations into small functions. Please don't be upset with that very simplistic
statement if you are a functional programming expert. :)

Can the be decomposed any more? Sure it can.

    :::javascript
    require(["rajo.util"], function(Util) {
        var sum = function(a, b) { return window.parseInt(a) + window.parseInt(b); };
        var total = Util.reduce(0, data.seriesTotals, sum);

        // total == 1413
    });

With this example we now have a reusable **sum()** function. It is simple, reuable
and pure. By pure I mean that it takes a series of inputs and returns an output
without any side effects, and that it is deterministic. By deterministic I mean
that given the same inputs it will always produce the same output.

I am enjoying what little I am learning and applying in functional programming, and
I look forward to learning more. Happy coding!