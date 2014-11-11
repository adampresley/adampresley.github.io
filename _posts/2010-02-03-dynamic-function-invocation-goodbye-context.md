---
layout: post
title: Dynamic Function Invocation - Goodbye Context!
date: 2010-02-03 04:57:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: dynamic-function-invocation-goodbye-context
---

Last night I posted about a small, unscientific performance test of
various method for dynamically invoking a method. So, for example, if
you have a method name stored dynamically somewhere (like a database, or
XML document) to be invoked against some component, there are a couple
of ways to do it. I demonstrated that one can create a variable
reference to a method on a component using evaluate, and then
continually invoke the method against the reference.  
  
> You should be aware however, that while CF functions are first   
> class citizens, they dont retain their context.  
  
However, as Mr. Mark Mandel and Mr. Ben Nadel pointed out above I should
be careful because the way ColdFusion actually processes CFC methods,
creating a "reference" to a method on a component, and invoking it that
way means the method loses, or more appropriately changes context. Allow
me to illustrate.   
  
Let's start with a simple CFC that has two methods. The first method is
a standard **init** function to setup the context of our component.
In here we will setup a private variable called **multiplier**. We
then have a method called **doTest** which accepts a single
argument, *value*, and returns the product of *value* and
*multiplier*. Let's see that now.  
  
{% highlight cfm %}
<cfcomponent displayname="Test" output="false">

	<cffunction name="init" access="public" output="false">
		<cfset variables.multiplier = 10 />
		<cfreturn this />
	</cffunction>

	<cffunction name="doTest" access="public" output="false">
		<cfargument name="value" required="false" />
		<cfreturn value * variables.multiplier />
	</cffunction>

</cfcomponent>
{% endhighlight %}

Simple enough. Now let's look at using the same method I did last night,
and see what happens.   

{% highlight cfm %}
<cfset rc.object = createObject("component", "com.test").init() />
<cfset rc.functionName = "doTest" />
<cfset rc.ptr = evaluate("rc.object.#rc.functionName#") />
<cfset rc.result = rc.ptr(5) />

<cfdump var="#rc#" label="Test Results" />
{% endhighlight %}
  
**Kaboom!**   
  
Why? I cannot say with any real certainty, except that it seems
ColdFusion seems to wrap CFCs in a sort of proxy, each function becoming
a Java class, and the proxy simply instantiates and invokes the
requested methods. When creating this function reference it seems we
lose this proxy container, and thus context (which contains all the
necessary scopes).   
  
I say all of that of course only through observation of the call stack,
and not much more. Perhaps someone with a bit more knowledge on this
subject can shed some light?
