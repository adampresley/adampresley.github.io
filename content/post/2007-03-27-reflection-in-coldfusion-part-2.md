---
author: "Adam Presley"
date: 2007-03-27T03:26:00Z
tags: ["development", "coldfusion", "reflection"]
title: "Reflection in ColdFusion Part 2"
slug: "reflection-in-coldfusion-part-2"
---

Yesterday I was working on using [Java reflection to create function
pointers in ColdFusion](http://blog.adampresley.com/2007/03/27/reflection-in-coldfusion/) to native ColdFusion methods. I was able to
get this to work on any function with only String parameters, but not
integer. Well, I found out why.

The integer data type in Java is a primitive data type. There **IS** a
class representation but it still maps down to the primitive data type.
This posed a problem trying to point to a ColdFusion function such as
*FindNoCase*. Why? Cause it takes two strings and one integer argument,
and the *java.lang.Integer* class wasn't doing the trick.

Turns out that the primitive wrapper classes have a property called
*TYPE* that return an instance of the **primitive type**. THIS was
exactly what I needed. So, here is an example of creating a function
pointer to *FindNoCase*.

{{< highlight cfm >}}
<cfset stringFunc = createObject("java", "coldfusion.runtime.StringFunc")>
<cfset stringClass = createObject("java", "java.lang.String")>
<cfset integerClass = createObject("java", "java.lang.Integer")>

<!--- Setup the argument types --->
<cfset argumentTypeArray[1] = stringClass.getClass()>
<cfset argumentTypeArray[2] = stringClass.getClass()>
<cfset argumentTypeArray[3] = integerClass.TYPE>

<cfset method = stringFunc.getClass().getMethod("FindNoCase", argumentTypeArray)>

<!--- Call the function. We must pass arguments as an array of objects. --->
<cfset searchWhat = "This is a test.">
<cfset searchFor = "is">
<cfset startPos = 0>

<cfset argumentArray = arrayNew(1)>
<cfset argumentArray[1] = searchFor>
<cfset argumentArray[2] = searchWhat>
<cfset argumentArray[3] = javaCast("int", startPos)>

<!--- Invoke the function, and show some results --->
<cfset result = method.invoke(stringFunc, argumentArray)>

<cfoutput>

<fieldset>

<legend>FindNoCase Example</legend>
Search What: #searchWhat#
Search For: #searchFor#
Start At: #startPos#

result = #result#
</fieldset>


</cfoutput>
{{< / highlight >}}

And voila! It works!
