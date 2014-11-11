---
layout: post
title: Intro to DSLs in Groovy, Part 2
date: 2011-03-24 09:09:00
author: Adam Presley
status: Published
tags: development groovy dsl
slug: intro-to-dsls-in-groovy-part-2
---

In my [previous post](|filename|/intro-to-dsls-in-groovy-part-1.md) I introduced a simple recipe DSL which provided
a really easy way to describe a recipe, its ingredients and
instructions. In part two I will start the process of how we can make
this tiny "language" of ours execute using Groovy's excellent ability to
throw out the syntax noise to make our "language" seem more natural.

To get started we'll need to define the foundation for our DSL. If you
recall the first line of the DSL code it looks like this:

{% highlight groovy %}
Recipe.create "Creamy Mac n' Cheese", {
{% endhighlight %}

To accomplish this we are going to create a class, as well as a few
variables to hold the information about our recipe. The first method
called is **create**, which will be a static method that uses a little
closure magic to execute the DSL code after the first curly brace. This
static method **create** will create a copy of the passed in closure,
change the delegate to a new **Recipe** object with the passed in name,
then execute. Changing the delegate tells the closure who it is
executing **for**. Yes, it's magic.

{% highlight groovy %}
class Recipe {
   def name = ""
   def ingredients = []
   def instructions = []

   static def create(String name, Closure c) {
      def clone = c.clone()
      clone.delegate = new Recipe(name: name)
      clone()
   }
}
{% endhighlight %}

Now our **Recipe** class needs methods to add ingredients and
instructions. These methods are simple as they will just push what is
send to them to the two collections **ingredients** and
**instructions**.

{% highlight groovy %}
void addIngredient(def measurement, String ingredient) {
   ingredients << [ measure: measurement, ingredient: ingredient ]
}

void addInstructions(String... instruction) {
   instruction.each { instructions << it }
}
{% endhighlight %}

To add ingredients to the recipe we take two arguments: measurement, and
the ingredient. The **Measurement** class is something we will get to
soon. This code is easy as it just adds the two items as a Map to the
**ingredients** collection. The **addInstructions** method is similar.
Notice the "..." after the **String** type. That indicates we will have
a list with an unknown number of elements being passed in. This allows a
list of instruction argument strings to be passed in, and it will get
treated as a collection. The whole listing looks like this so far.

{% highlight groovy %}
class Recipe {
   def name = ""
   def ingredients = []
   def instructions = []

   static def create(String name, Closure c) {
      def clone = c.clone()
      clone.delegate = new Recipe(name: name)
      clone()
   }

   void addIngredient(def measurement, String ingredient) {
      ingredients << [ measure: measurement, ingredient: ingredient ]
   }

   void addInstructions(String... instruction) {
      instruction.each { instructions << it }
   }
}
{% endhighlight %}

What we can do with what we have now is something like this:

{% highlight groovy %}
Recipe.create "Creamy Mac n' Cheese", { 
   addIngredient 1, "Mac n' Cheese"
   addIngredient 1, "Hamburger meat"
   addIngredient 1, "Cream of Mushroom"

   addInstructions "Brown hamburger meat",
      "Bring mac n' cheese to a boil",
      "Drain water from noodles",
      "Stir in cheese mixture",
      "Add cream of mushroom and meat",
      "Stir",
      "Eat"
}
{% endhighlight %}

If you compare that example to the one from the first post you will see
we are getting close. Up next we will talk about the **Measurement**
class, and how to use Groovy's metaclass enhancements to define custom
measurement types against numbers. Stay tuned!
