---
author: "Adam Presley"
date: 2011-03-22T18:40:00Z
tags: ["development", "groovy", "dsl"]
title: "Intro to DSLs in Groovy, Part 1"
slug: "intro-to-dsls-in-groovy-part-1"
---

Here in a couple of weeks I will be presenting at the Dallas/Ft. Worth
Groovy User Group meeting on an introduction to DSLs, or *Domain
Specific Languages*, in Groovy. What is particularly fun about this is
how wonderfully easy Groovy makes this task. In this post I am basically
giving a sneak peak into my presentation. Arent I nice? :)

What is a DSL, and why do you care? The simple version is a DSL is a
small, focused language designed to make the description and execution
of some task in a specific domain easier. In fact we all use DSLs every
day and completely take for granted that we do. SQL, HTML, CSS, an Ant
XML file all DSLs. All designed to do something specific for the domain
they are related to.

Here in Part 1 of this series I will show what our language will look
like. This little "language" will allow us to easily define a food
**recipe**. Once defined we can easily tell this code to display the
recipe card, or even convert the recipe to XML. Here is what the
language will look like, as well as the rendered XML output.

## Example
{{< highlight groovy >}}
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
{{< / highlight >}}

## XML Output
{{< highlight xml >}}
<recipe name='Creamy Mac n&apos; Cheese'>
   <ingredients>
      <ingredient>&lt;![CDATA[1 box of Mac n' Cheese]]&gt;</ingredient>
      <ingredient>&lt;![CDATA[1 pound of Hamburger meat]]&gt;</ingredient>
      <ingredient>&lt;![CDATA[1 can of Cream of Mushroom]]&gt;</ingredient>
   </ingredients>
   <instructions>
      <instruction>&lt;![CDATA[Brown hamburger meat]]&gt;</instruction>
      <instruction>&lt;![CDATA[Bring mac n' cheese to a boil]]&gt;</instruction>
      <instruction>&lt;![CDATA[Drain water from noodles]]&gt;</instruction>
      <instruction>&lt;![CDATA[Stir in cheese mixture]]&gt;</instruction>
      <instruction>&lt;![CDATA[Add cream of mushroom and meat]]&gt;</instruction>
      <instruction>&lt;![CDATA[Stir]]&gt;</instruction>
      <instruction>&lt;![CDATA[Eat]]&gt;</instruction>
   </instructions>
</recipe>
{{< / highlight >}}

Upon first glance it should be pretty apparent what we are trying to do.
We first define a recipe with a name. Then we start adding ingredients,
each with a specific measurement. Notice each measurement is different,
and quite versatile. From here we add a list of instructions, and
finally display the results in XML.

But how do we define such a language? Historically one might find
themselves investigating compiler compilers, writing nasty BNF language
descriptions, and maybe even writing parsers by hand. You might even
start Googling Recursive-Descent Parsers and RPN expression parsing
notation. Luckily, with the Groovy language, you dont have to worry
about that too much.

Tune in next time for how we will get started defining this language.
