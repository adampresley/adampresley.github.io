---
layout: post
title: Comparing Classes in Python
date: 2013-05-22 22:24:00
author: Adam Presley
status: Published
tags: python
slug: comparing-classes-in-python
---

Most object oriented languages have a means to compare two objects or
classes. Java has the **Comparator<T>** interface. In C# you can
override the **Equals()** method. Python naturally has a way to make
your class comparable by using a magic method named **__eq__**. It
takes an argument of the object you are comparing against, as well as
the standard reference to *self*. In it you return *True* or *False* to
indicate if the object passed in is equal to itself. Here's a sample.  
  
{% highlight python %}
class Sample():
   def __init__(self):
      self.name = ""
      self.age = 0

   def __eq__(self, other):
      return (self.name == other.name and 
       self.age == other.age)
{% endhighlight %}

The above class has a constructor method as defined by **__init__**
that sets up two properties named *name* and *age*. It then defines the
**__eq__** magic method. This is called automatically when you
attempt to compare the equality of two objects of type **Sample** like
so.  

{% highlight python %}
a = Sample()
a.name = "Bob"
a.age = 11

b = Sample()
b.name = "Julia"
b.age = 12

result = (a == b)
print result
{% endhighlight %}

The result of this comparison will be *False* as the two objects do not
have the same name and age properties. And that is how you compare class
objects in Python. Cheers, and happy coding!
