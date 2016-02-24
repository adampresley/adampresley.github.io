---
author: "Adam Presley"
comments: true
date: 2014-11-10T14:39:00Z
tags: ["development", "golang"]
title: "A Go Weekend Puzzler"
slug: "a-go-weekend-puzzler"
---

[Adam Cameron](http://blog.adamcameron.me/) posted a code challenge [on his blog](http://blog.adamcameron.me/2014/11/something-for-weekend-wee-code-quiz-in.html) this last Friday. I have provided my answer in the form of a [Google Go project](https://github.com/adampresley/adamCameronCodeChallenge201411). For those interested a 10,000ft view of how this works can be found below. Here are the instructions he provided.

> For a given array and a given threshold, return the subarray which contains the longest run of consecutive numbers which - in total - are equal-to or less than the threshold.

* The main program is a console application that accepts two arguments:
    - input - A comma-delimited list of integer numbers
    - threshold - The total value an array sequence may not exceed
* A **Scheduler** component (probably a bad name) starts slicing the input integer array into sub-sequences
* Each sub-sequence is summed in a *goroutine*
* The sub-sequence is totaled, and both the sub-sequence and total are put into a structure that is written to a channel
* Another *goroutine* listens for writes to this channel and stuffs the structures into a result array
* The result array is traversed to determine which sub-sequence is longest.
    - If there is a tie for longest the sub-sequence with the greatest total sum value is chosen

I chose to make the slicing and summing activity multi-process mostly for the fun of the exercise. Though in theory one could provide a very large set of integers and it would process rather quickly I suspect. Either way this was a fun exercise. You should certainly go check out the blog post and see other people's submissions. There are entries in language including Clojure, CFML, PHP and JavaScript.