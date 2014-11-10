---
layout: post
title: Using Lambda for Sorting in C# .NET
date: 2010-12-07 23:49:00
author: Adam Presley
status: Published
tags: development c#
slug: using-lambda-for-sorting-in-c-net
---

Here's a quick little tidbit that helped me this week. While writing a
C# Windows application I found myself with a **List** of objects that
did not implement a comparator interface. Because of this when I tried
to call **mylist.Sort()** I got an error complaining about how it
couldn't compare the two objects.

The quick and dirty way to address this is C# .NET 3.5 and above is to
use a Lambda expression. So given a list named **mylist**, here is how
one can do this.

	:::c#
	mylist.Sort( (a, b) => a.ToString().CompareTo(b.ToString()) );

Notice how we start immediately with a set of arguments, followed by
**=>**. This is the start of the lambda expression, and it tells us
"arguments a and b goes to {some expression}". In our case **{some
expression}** is to take the string version of object *a* and compare it
to the string version of object *b*. This works because the objects in
question **did** override the **ToString()** method allowing us to do a
basic string comparison between the two objects.

Pretty nifty eh? Happy coding!
