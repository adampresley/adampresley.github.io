---
layout: post
title: ColdFusion Exception Handling
date: 2006-12-06 15:41:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: coldfusion-exception-handling
---

For the last few days I've been messing with ColdFusion exception
handling, specifically on the TRY/CATCH block side. At first I was
greatly intrigued by the fact that you could throw exceptions based off
Java classes derived from the *RuntimeException*class. That would look
something like this.  

{% highlight coldfusion %}
<cfset illegalArgument = CreateObject("java", "java.lang.IllegalArgumentException").init("Bad argument dude!")>

<cftry>
	<!--- Do some code here. Throw a fake exception for demonstration purposes --->
	<cfif 1 NEQ 2><cfthrow object="#illegalArgument#"></cfif>

<cfcatch type="Any">
	<cfdump var="#CFCATCH#">
</cfcatch>
</cftry>
{% endhighlight %}

This code instantiates an object of type *IllegalArgumentException* and
throws an error upon a test condition failure (albeit a false one). This
may not seem like much on the surface but what this means is that you
can create your own custom exception classes to handle conditions where
an error should occur, and you'd like execution flow to cease and be
rerouted.  
  
Let's say that you wanted to throw an error if a phone number posted to
the page wasn't formatted like *999-999-9999*. Let us also say that you
wanted to create a custom error class in Java to be thrown on this
condition. Here is what the java class, and the code to use it would
look like.  
  
{% highlight java %}
import java.lang.RuntimeException;
import java.lang.String;

public class myPhoneNumberFormatException extends RuntimeException {
	public myPhoneNumberFormatException() { super(); }
	public myPhoneNumberFormatException(String message) { super(message); }
}
{% endhighlight %}

To use it...  

{% highlight coldfusion %}
<cfset phoneNumberFormatException = CreateObject("java", "myPhoneNumberFormatException").init("Invalid phone number format!")>

<cftry>
	<cfif NOT phoneNumber.trim().matches('[0-9]{3}-[0-9]{3}-[0-9]{4}'>
		<cfthrow object="#phoneNumberFormatException#">
	</cfif>

<cfcatch type="Any">
	<cfdump var="#CFCATCH#">
</cfcatch>
</cftry>
{% endhighlight %}

After going through all this, though, I began to question if there was
any added benefit into diving into the Java side versus just using a
technique of defining exception error codes and messages in a structure,
for example. Then, after trying this, I also realized that ColdFusion
provides the execution stack to you after an exception has been thrown.
Perfect!  
  
So at this point I was pretty happy... But then I discovered that THROW
and RETHROW **DO NOT EXIST IN CFSCRIPT!!!** Oh... my... gawd... BAH!!  
  
Aside from that last little disappointment I am quite happy with the
mechanisms that ColdFusion provides for TRY/CATCH exception handling.
