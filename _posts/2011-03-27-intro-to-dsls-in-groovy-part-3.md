---
layout: post
title: Intro to DSLs in Groovy, Part 3
date: 2011-03-27 11:45:00
author: Adam Presley
status: Published
tags: development groovy
slug: intro-to-dsls-in-groovy-part-3
---

Today let's continue our series of getting started with simple DSL
creation in Groovy. In [part 1](|filename|/intro-to-dsls-in-groovy-part-1.md) we talked about the *what* and *why*
of Domain Specific Langages, and what our sample DSL would look like.
Then in [part 2](|filename|/intro-to-dsls-in-groovy-part-2.md) we actually started building the code in Groovy
creating the class necessary to make the DSL code start to work. In part
3 we will enhance our code to allow us to add ingredients with
measurements. Adding this will make the DSL more fluent and readable,
which is one of the main reasons you would bother to craft a DSL to
begin with!

If you've read my blog before you will have seen me talk about and
demonstrate how one can expand the capabilities of the existing Groovy
language though metaClass enhancements. I hope you were paying attention
because we are going to use them again. For our purposes we want to
enhance all number classes to be able to specify a unit of measurement
against it, which we will use later when displaying our recipe in
various output formats. So, as an example if our recipe called for 1 and
half cups of cheese, it could be written like **1.5.cups**. We also wish
to ensure that all number classes get this, so our metaClass enhancement
needs to apply down the hierarchy. When a unit of measurement is applied
to a number, instead of returning an actual number class, we will
instead create a new Measurement class that will be returned in its
place.

{% highlight groovy %}
/*
 * Ensure that changes to base classes through metaClass
 * expand downwards.
 */
ExpandoMetaClass.enableGlobally()


/**
 * Defines a unit and amount of measurement. Used for ingredients
 */
class Measure {
   def amount
   def unit

   String toString() {
      "${amount} ${unit}"
   }
}


/*
 * Allow all number classes to be expressed as #.measurement
 */
Number.metaClass.getProperty = { String measurement -> 
   new Measure(amount: delegate, unit: measurement) 
}
{% endhighlight %}

Now with the measurement enhancement in place our DSL can now be written
like this.

{% highlight groovy %}
Recipe.create "Creamy Mac n' Cheese", { 
   addIngredient 1.box, "Mac n' Cheese"
   addIngredient 1.pound, "Hamburger meat"
   addIngredient 1.can, "Cream of Mushroom"

   addInstructions "Brown hamburger meat",
      "Bring mac n' cheese to a boil",
      "Drain water from noodles",
      "Stir in cheese mixture",
      "Add cream of mushroom and meat",
      "Stir",
      "Eat"
}
{% endhighlight %}

Now, finally, let's create a couple of methods in our **Recipe** class
from part 2 that will display our culinary creations in a couple of
different ways. We will make code to show the recipe in a standard "1,
2, 3" step format, and also in XML. To make the DSL syntax a little more
fluent we will also create some enums that we will statically import to
be used as "constants" to describe how to display our recipe.

{% highlight groovy %}
import static ShowActions.*
import groovy.xml.MarkupBuilder

/*
 * These are the various actions that the "show" command can perform.
 */
enum ShowActions { RecipeCard, Ingredients, Instructions, RecipeXml }


class Recipe {
   def name = ""
   def ingredients = []
   def instructions = []

   static def create(String name, Closure c) {
      def clone = c.clone()
      clone.delegate = new Recipe(name: name)
      clone.resolveStrategy = Closure.DELEGATE_ONLY    // Resolve calls in this closure internally only
      clone()
   }

   void addIngredient(def measurement, String ingredient) {
      ingredients << [ measure: measurement, ingredient: ingredient ]
   }

   void addInstructions(String... instruction) {
      instruction.each { instructions << it }
   }

   void show(ShowActions action) {
      def printIngredients = {
      println "INGREDIENTS:"
      ingredients.eachWithIndex { item, index -> println "${index + 1}. ${item.measure} of  ${item.ingredient}" }
   }

      def printInstructions = {
         println "

DIRECTIONS:"
         instructions.eachWithIndex { item, index -> println "${index + 1}. ${item}" }
      }

      switch (action) {
         case ShowActions.RecipeCard:
            printIngredients()
            printInstructions()
            break

         case ShowActions.Ingredients:
            printIngredients()
            break

         case ShowActions.Instructions:
            printInstructions()
            break

         case ShowActions.RecipeXml:
            println toXml()
            break
      }
   }

   def toXml() {
      def writer = new StringWriter()
      def xml = new MarkupBuilder(writer)

      xml.recipe(name: this.name) {
         ingredients {
            ingredients.each { ing ->
               ingredient { mkp.yieldUnescaped "&lt;![CDATA[${ing.measure} of ${ing.ingredient}]]&gt;" }
            }
         }
         instructions { 
            instructions.each { ins ->
               instruction { mkp.yieldUnescaped "&lt;![CDATA[${ins }]]&gt;" }
            }
         }
      }

      writer.toString()
   }
}
{% endhighlight %}

The code above is the **Recipe** class with two new methods: **show()**
and **toXml()**. Here are a couple of sample uses with these new
methods.

## Recipie XML
{% highlight groovy %}
Recipe.create "Creamy Mac n' Cheese", { 
   addIngredient 1.box, "Mac n' Cheese"
   addIngredient 1.pound, "Hamburger meat"
   addIngredient 1.can, "Cream of Mushroom"

   addInstructions "Brown hamburger meat",
      "Bring mac n' cheese to a boil",
      "Drain water from noodles",
      "Stir in cheese mixture",
      "Add cream of mushroom and meat",
      "Stir",
      "Eat"

   show RecipeXml
}
{% endhighlight %}

## Recipie Card
{% highlight groovy %}
Recipe.create "Creamy Mac n' Cheese", { 
   addIngredient 1.box, "Mac n' Cheese"
   addIngredient 1.pound, "Hamburger meat"
   addIngredient 1.can, "Cream of Mushroom"

   addInstructions "Brown hamburger meat",
      "Bring mac n' cheese to a boil",
      "Drain water from noodles",
      "Stir in cheese mixture",
      "Add cream of mushroom and meat",
      "Stir",
      "Eat"

   show RecipeCard
}
{% endhighlight %}

I hope this small introduction into DSL-world has been interesting. This
example barely scratches the surface of what can be done, and there is
certainly some pretty heavy reading out there on what can be done, and
how one can make DSLs. Here are some links to recommended reading.

-   [Writing Domain-Specific Languages](http://docs.codehaus.org/display/GROOVY/Writing+Domain-Specific+Languages)
-   [A Groovy DSL from scratch in 2 hours](http://groovy.dzone.com/news/groovy-dsl-scratch-2-hours)
-   [A Text Adventure DSL in Groovy](http://blogs.citytechinc.com/sanderson/?p=92)
-   Anything Martin Fowler!

Here is the code in full:

{% highlight groovy %}
import static ShowActions.*
import groovy.xml.MarkupBuilder

/*
 * These are the various actions that the "show" command can perform.
 */
enum ShowActions { RecipeCard, Ingredients, Instructions, RecipeXml }


/*
 * Ensure that changes to base classes through metaClass
 * expand downwards.
 */
ExpandoMetaClass.enableGlobally()


/**
 * Defines a unit and amount of measurement. Used for ingredients
 */
class Measure {
   def amount
   def unit

   String toString() {
      "${amount} ${unit}"
   }
}


/**
 * Defines the recipe "language", it's data, and transformations.
 */
class Recipe {
   def name = ""
   def ingredients = []
   def instructions = []

   static def create(String name, Closure c) {
      def clone = c.clone()
      clone.delegate = new Recipe(name: name)
      clone.resolveStrategy = Closure.DELEGATE_ONLY    // Resolve calls in this closure internally only
      clone()
   }

   void addIngredient(def measurement, String ingredient) {
      ingredients << [ measure: measurement, ingredient: ingredient ]
   }

   void addInstructions(String... instruction) {
      instruction.each { instructions << it }
   }

   void show(ShowActions action) {
      def printIngredients = {
      println "INGREDIENTS:"
      ingredients.eachWithIndex { item, index -> println "${index + 1}. ${item.measure} of  ${item.ingredient}" }
   }

      def printInstructions = {
         println "

DIRECTIONS:"
         instructions.eachWithIndex { item, index -> println "${index + 1}. ${item}" }
      }

      switch (action) {
         case ShowActions.RecipeCard:
            printIngredients()
            printInstructions()
            break

         case ShowActions.Ingredients:
            printIngredients()
            break

         case ShowActions.Instructions:
            printInstructions()
            break

         case ShowActions.RecipeXml:
            println toXml()
            break
      }
   }

   def toXml() {
      def writer = new StringWriter()
      def xml = new MarkupBuilder(writer)

      xml.recipe(name: this.name) {
         ingredients {
            ingredients.each { ing ->
               ingredient { mkp.yieldUnescaped "&lt;![CDATA[${ing.measure} of ${ing.ingredient}]]&gt;" }
            }
         }
         instructions { 
            instructions.each { ins ->
               instruction { mkp.yieldUnescaped "&lt;![CDATA[${ins }]]&gt;" }
            }
         }
      }

      writer.toString()
   }
}

/*
 * Allow all number classes to be expressed as #.measurement
 */
Number.metaClass.getProperty = { String measurement -> 
   new Measure(amount: delegate, unit: measurement) 
}


/*
 * Actually try USING our little DSL.
 */
Recipe.create "Creamy Mac n' Cheese", { 
   addIngredient 1.box, "Mac n' Cheese"
   addIngredient 1.pound, "Hamburger meat"
   addIngredient 1.can, "Cream of Mushroom"

   addInstructions "Brown hamburger meat",
      "Bring mac n' cheese to a boil",
      "Drain water from noodles",
      "Stir in cheese mixture",
      "Add cream of mushroom and meat",
      "Stir",
      "Eat"

   show RecipeXml
}
{% endhighlight %}

Happy coding!
