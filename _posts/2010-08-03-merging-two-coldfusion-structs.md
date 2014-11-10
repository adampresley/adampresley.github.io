---
layout: post
title: Merging Two ColdFusion Structs
date: 2010-08-03 06:19:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: merging-two-coldfusion-structs
---

Here's a quick little tidbit. My coworker [Steve Good](http://stevegood.org) asked me if I
knew of a quick way to merge two ColdFusion structures together, kind of
like how jQuery has the **$.extend()** method. Well there
**is** in fact a way to do this! And it's super easy.  

Let's say you have structure one that has two keys, *firstName* and
*lastName*. 
  
    :::coldfusion
    <cfset struct1 = { firstName = "Adam", lastName = "Presley" } />
  
And now we have structure two that has two keys, *firstName* and
*age*.  

    :::coldfusion
    <cfset struct2 = { firstName = "Michael", age = 33 } />

Using a nifty ColdFusion method we can mash the two together in a single
line of code.  

    :::coldfusion
    <cfset structAppend(struct1, struct2) /> 
    <!--- <cfset struct1.putAll(struct2) /> ---> 
  
Woah, that was easy! The end result will be a structure that would look
like this.  

    :::coldfusion
    <cfset struct1 = { firstName = "Michael", lastName = "Presley", age = 33 } />
  
Notice the commented out version. That is the underlying method to do
the same thing as StructAppend. The benefit? Nothing I can think of. :)  
  
Enjoy, and happy coding!
