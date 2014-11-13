---
layout: post
title: Removing Duplicate Array Items in ColdFusion
date: 2009-10-22 07:41:00
author: Adam Presley
status: Published
tags: development coldfusion java
slug: removing-duplicate-array-items-in-coldfusion
---
If you've ever had the need to quickly remove duplicate items in an
array you might find yourself resorting to the old loop method. This
involves looping over the input array, then further looping over a
result array, checking to see if you've already inserted the input
value, and if not, inserting it. That's a bit of looping and the code is
not quite a pretty or fun! Here's what that would look like.

{% highlight cfm %}
<cffunction name="removeDuplicatesFromArrayLoop" returntype="array" access="public" output="false">
	<cfargument name="input" type="array" required="true" />

	<cfset var result = [] />
	<cfset var outer = 0 />
	<cfset var inner = 0 />
	<cfset var found = false />

	<cfloop array="#arguments.input#" index="outer">
		<cfloop array="#result#" index="inner">
			<cfif outer EQ inner>
				<cfset found = true />
			</cfif>
		</cfloop>

		<cfif !found>
			<cfset arrayAppend(result, outer) />
		</cfif>

		<cfset found = false />
	</cfloop>

	<cfreturn result />
</cffunction>
{% endhighlight %}

As you can see we have an outer and inner loop. The outer going over the
input, and the inner iterating over the resulting array that should
contain our original input minus the duplicate values. A fine solution
no doubt.

I would like to get a little more sophisticated, however, and take
advantage of the fact that ColdFusion is built on top of Java. The Java
method relies on two class: [java.util.LinkedHashSet](http://java.sun.com/javase/6/docs/api/java/util/LinkedHashSet.html), and
[java.util.List](http://java.sun.com/javase/6/docs/api/java/util/List.html). First we use a built in method of a ColdFusion array
called *subList*. This is available due to the fact that ColdFusion
arrays are built on top of the [java.util.Vector](http://java.sun.com/javase/6/docs/api/java/util/Vector.html) class. This method
will give us a **java.util.List** object. We then need to feed that List
to the **java.util.LinkedHashSet**.

The **java.util.LinkedHashSet** is a doubly-linked list, and as such
allows bi-directional iteration, and it also respects insertion order.
This means that the items in the LinkedHashSet will be in the same order
as they were inserted into your original input array, which is
desirable. It also does not allow duplicate entries in its list, and
thus will reject inserting any items that it already contains. This
gives us the removing of duplicates functionality.

From here we then create a new array called result. Once again we delve
to the Java methods and call the *addAll* method, which takes an
argument of type [java.util.Collection](http://java.sun.com/javase/6/docs/api/java/util/Collection.html). It just so happens that the
**java.util.LinkedHashSet** implements the **java.util.Collection**
interface, so we will pass that on to the *addAll* method, which in turn
will take all the elements from the LinkedHashSet and put them into our
new array.

Let's take a look at a function to do this.

{% highlight cfm %}
<cffunction name="removeDuplicatesFromArray" returntype="array" access="public" output="false">
	<cfargument name="input" type="array" required="true" />
	<cfargument name="sortArray" type="boolean" required="false" default="false" />
	<cfargument name="sortType" type="string" required="false" default="textnocase" />
	<cfargument name="sortDirection" type="string" required="false" default="asc" />

	<cfscript>

		var list = arguments.input.subList(0, arguments.input.size());
		var set = createObject("java", "java.util.LinkedHashSet").init(list);
		var result = [];

		result.addAll(set);

		if (arguments.sortArray) {
			arraySort(result, arguments.sortType, arguments.sortDirection);
		}

		return result;

	</cfscript>
</cffunction>
{% endhighlight %}

As you can see the code is pretty simple, though the explanation is
lengthy. And as an added bonus I've added arguments that allow you to
optionally sort the array once it has removed duplicates. It uses the
good old ColdFusion method *arraySort* to do this. Below is a sample
demonstrating these functions being used. Happy coding!

{% highlight cfm %}
<cffunction name="removeDuplicatesFromArray" returntype="array" access="public" output="false">
	<cfargument name="input" type="array" required="true" />
	<cfargument name="sortArray" type="boolean" required="false" default="false" />
	<cfargument name="sortType" type="string" required="false" default="textnocase" />
	<cfargument name="sortDirection" type="string" required="false" default="asc" />

	<cfscript>

		var list = arguments.input.subList(0, arguments.input.size());
		var set = createObject("java", "java.util.LinkedHashSet").init(list);
		var result = [];

		result.addAll(set);

		if (arguments.sortArray) {
			arraySort(result, arguments.sortType, arguments.sortDirection);
		}

		return result;

	</cfscript>
</cffunction>

<cffunction name="removeDuplicatesFromArrayLoop" returntype="array" access="public" output="false">
	<cfargument name="input" type="array" required="true" />

	<cfset var result = [] />
	<cfset var outer = 0 />
	<cfset var inner = 0 />
	<cfset var found = false />

	<cfloop array="#arguments.input#" index="outer">
		<cfloop array="#result#" index="inner">
			<cfif outer EQ inner>
				<cfset found = true />
			</cfif>
		</cfloop>

		<cfif !found>
			<cfset arrayAppend(result, outer) />
		</cfif>

		<cfset found = false />
	</cfloop>

	<cfreturn result />
</cffunction>


<cfset a1 = [ "value1", "value2", "value1", "value4", "value3", "value1" ] />

<cfoutput>

<h1>LOOP WAY:</h1>

<strong>Original Array:</strong>
<cfdump var="#a1#" />

<cfset a1NoDuplicates = removeDuplicatesFromArrayLoop(a1) />

<strong>Duplicates Removed Array:</strong>
<cfdump var="#a1NoDuplicates#" />


<h1>JAVA WAY:</h1>

<strong>Original Array:</strong>
<cfdump var="#a1#" />

<cfset a1NoDuplicates = removeDuplicatesFromArray(a1) />

<strong>Duplicates Removed Array:</strong>
<cfdump var="#a1NoDuplicates#" />

</cfoutput>
{% endhighlight %}
