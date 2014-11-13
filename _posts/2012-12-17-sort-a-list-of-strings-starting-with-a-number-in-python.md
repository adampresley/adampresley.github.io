---
layout: post
title: Sort a List of Strings Starting With a Number in Python
date: 2012-12-17 13:47:00
author: Adam Presley
status: Published
tags: development python
slug: sort-a-list-of-strings-starting-with-a-number-in-python
---
I recently ran into a situation where had a list of strings I wanted
sorted in a particular way in Python. The **sorted()** method offers a
quick way to sort a collection, but in my case I needed to do more than
just sort alphabetically.

The following is a partial example of a list of strings that represent a
table of contents that need sorting.

{% highlight python %}
items = [
	"10. Chapter 10",
	"11. Chapter 11",
	"8. Chapter 8",
	"9. Chapter 9",
]
{% endhighlight %}

The above shows the strings out of order for demonstration purposes. In
my case the list is populated from a call to **os.listdir()** and came
in being out of order to start with.

To address this issue the Python **sorted()** method supports the
ability to define a key to sort against. Furthermore that key can be a
lambda expression. This means that I can somehow parse the incoming
string to get only the numeric part at the beginning and use *that* as
the key to sort on.

{% highlight python %}
items = sorted(items, key=lambda item: int(item.partition(".")[0]))
{% endhighlight %}

The magic is in two parts here. The first is the lambda expression. This
defines an inline function used to provide the **sorted()** method a key
to sort on. Next I mentioned that we need to get only the numeric part
of the string. The **partition()** method of the string object in Python
will split a string up based on the first occurrence of **"."** found in
the string. Then we cast the number found as an integer, and we have a
key. Tada!

Hope this was helpful. Happy coding!
