---
layout: post
title: Roman Enumeration
date: 2012-04-03 13:05:00
author: Adam Presley
status: Published
tags: development python
slug: roman-enumeration
---

While trolling The Daily WTF I came across a post called [Roman
Enumeration](http://thedailywtf.com/Articles/Roman-Enumeration.aspx). It is basically
about a shift to HR for hiring developers and the code produced by one such hire. In the article the
writer talks about giving new hires a training task to write a
program/function to convert a set of Roman numerals into decimal. It
then gives a snippet of one new hire's submission. Needless to say it is
... lacking. So I immediately thought it would be fun to make a quick
script to do this. Why? Distraction mostly. I needed a break. :) So
without further ado here is the Python code I wrote to do this.

{% highlight python %}
def convert(input):
    map = {
        "I": 1,
        "V": 5,
        "X": 10,
        "L": 50,
        "C": 100,
        "D": 500,
        "M": 1000
    }

    print "Input = %s" % (input)

    currentValue = 0
    result = 0
    index = 0

    while index < len(input):
        letter = input[index]
        currentValue = map[letter]
        nextValue = 0

        if (index < len(input) - 1):
            nextValue = map[input[index + 1]]

        if currentValue < nextValue:
            result += (nextValue - currentValue)
            index = index + 1
        else:
            result += currentValue

        index = index + 1

    return result

input = "MCMXLIV"
print convert(input)
{% endhighlight %}

Is there a more efficient way? I'm sure there is, but it was fun
nonetheless. Feel free to comment on how to make this better. Happy
coding!
