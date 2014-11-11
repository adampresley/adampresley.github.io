---
layout: post
title: ColdFusion Dynamic Function Invocation Methods
date: 2010-02-02 18:21:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: coldfusion-dynamic-function-invocation-methods
---

Tonight I was working with a project that a buddy and I are doing on the
side in an attempt to make money and I'm reviewing the code he's got so
far to make sure I have a good grasp on everything that's going on. He's
built his own ColdFusion framework, and it appears pretty solid, if not
a bit heavy. As I'm reviewing this code, however, I see that part of the
routing code (yes, it's an MVC framework) is using *evaluate* to
dynamically execute methods. Although I see **why** this is
happening, I've always assumed that *evaluate* was evil. My first
thought was "Why isn't he using CFINVOKE?". My second thought was "But
is that really better??"  
  
Being the curious guy that I am I decided to make a very unscientific
test where I would run two sets of code. The first set determines how
much time it takes to create an object using *createObject*, then call
a method against that object 100 times using *evaluate*. For example,  

{% highlight cfm %}
<cfset evaluate("someObject.#dynamicName#()") />
{% endhighlight %}

The second set determines how much time it takes to execute a method 100
times using just plain old CFINVOKE.  
  
I ran each test 10 times with the following results:  
  
* Evaluate: average of 267ms  
* CFINVOKE: average of 284ms  
  
The small margin of difference did catch me off guard, as I kind of
expected CFINVOKE to perform badly, and evaluate to do much better, even
though it is evil.  
  
Then I thought, "How about we create the object, then create a reference
to the method?" So something like this.  

{% highlight cfm %}
<cfset start = getTickCount() />
<cfset functionName = "test" />
<cfset o = createObject("MyObject").init() />
<cfset ptr = evaluate("l.o1.#functionName#") />
<cfloop from="1" to="100" index="l.i">
	<cfset ptr() />
</cfloop>

<cfset end = getTickCount() />  
{% endhighlight %}

I ran this 10 times, and got an average of 15.5ms. *WOAH!!!* Big
difference! So there ya go! Remember that ColdFusion is built on Java,
and references are a powerful feature! Happy coding!
