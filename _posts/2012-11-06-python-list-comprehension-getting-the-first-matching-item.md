---
layout: post
title: Python List Comprehension - Getting The First Matching Item
date: 2012-11-06 17:35:00
author: Adam Presley
status: Published
tags: development python
slug: python-list-comprehension-getting-the-first-matching-item
---

Today I learned a nifty trick in Python using list comprehensions. If
you don't know what list comprehensions are check out
<http://www.python.org/dev/peps/pep-0202/>.

I recently had a scenario where I wanted to find the first item in a
list that matched a single criteria. A basic loop can certainly do the
job, as show below. In this example I want to return the first item in
the list that is greater than the number 2.

{% highlight python %}
a = [1, 2, 3, 4, 5]

def findFirstNumberGreaterThan2():
   result = None

   for i in a:
      if i > 2:
         result = i
         break

   return result
{% endhighlight %}

This works fine and all, but it is what Python folks call "pythonic",
and there is certainly a more concise way to handle this problem. Enter
list comprehensions and the **next()** method. The **next()** method
retrieves the next item from an iterable (is that a word??) and returns
the value, or if it is not found the default value, found in the second
argument, is returned. Let's see this in action by reworking the
previous example.

{% highlight python %}
a = [1, 2, 3, 4, 5]

def findFirstNumberGreaterThan2():
   return next((i for i in a if i > 2), None)
{% endhighlight %}

Woah, that is pretty cool! The **next()** method takes two arguments.
The first being an object that can be iterated over, and the second
being a default value that is returned once the iterator is exhausted.
The second argument is optional.

I definitely need to spend more time digging into the "pythonic" tricks
of the trade. Happy coding!
