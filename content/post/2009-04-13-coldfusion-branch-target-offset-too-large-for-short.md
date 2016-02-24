---
author: "Adam Presley"
date: 2009-04-13T08:21:00Z
tags: ["development", "coldfusion"]
title: "ColdFusion - Branch Target Offset Too Large for Short"
slug: "coldfusion-branch-target-offset-too-large-for-short"
---

![JVM Error Screenshot](http://s3.amazonaws.com/www.adampresley.com/posts/jvmerror-branch-offset.jpg)

Yes, that is the interestingly ambiguous message I recieved today while
developing on a particularly large CFC. In this particular component
there was a function that was excessively long, so I decided to be cool
and break it apart a little bit. To start I took out a 523 line query
and moved it to a new function. When I then ran the code I recieved the
following error: "Branch target offset too large for short".

Ok... I've not quite heard of this one before, so I turned to the ever
trusty Google. Turns out that there is a limitation in the JVM where
branch statements (like GOTO) cannot reach an address (or offset) in the
code past 32,767 bytes, or a SHORT integer. In short (pun intended) one
of my functions, or some branch condition in an IF or SWITCH statement,
cannot be reached because it is at an offset that exceeds 32,767 bytes.
Code is too big!!

**UPDATE:** To fix this issue I moved a large branch of code that
contained a large number of IF/ELSE statements to a new function. This
apparently solved the issue and the code ran happily along again.
