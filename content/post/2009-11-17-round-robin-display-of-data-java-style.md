---
author: "Adam Presley"
date: 2009-11-17T11:21:00Z
tags: ["development", "coldfusion", "java"]
title: "Round Robin Display of Data - Java Style"
slug: "round-robin-display-of-data-java-style"
---

In my daily blog reading I came across [Mr. Ray Camden's blog post](http://www.coldfusionjedi.com/index.cfm/2009/11/17/Ask-a-Jedi-Round-robin-display-of-data)
about rotating a grouped set of data in a "round robin" fashion. In his
example he uses a multi-dimensional array, then uses ColdFusion to
iterate over the array, copy the item to move, delete it, them push it
to the end of the array, thus "rotating" it. That looks like this.

{{< highlight cfm >}}
<cfloop index="i" from="1" to="#arrayLen(data)#">
	<cfset movingItem = data[i][1]>
	<cfset arrayDeleteAt(data[i],1)>
	<cfset arrayAppend(data[i], movingItem)>
</cfloop>
{{< / highlight >}}

Being the giant nerd that I am I wanted to see if we could do this using
the big Java-Hemi under the hood of ColdFusion. Sure enough, the Java
[Collections](http://java.sun.com/j2se/1.4.2/docs/api/java/util/Collections.html)
class has a way to do this! Built right in! I'm not
going to go into the setup, because Mr. Camden took care of that already
([go read it](http://www.coldfusionjedi.com/index.cfm/2009/11/17/Ask-a-Jedi-Round-robin-display-of-data)),
but I will show you that we use the rotate method on the Collections class. And in his example
it is an array of arrays, so we will loop over the first level of the
array, and apply the [rotate method](http://java.sun.com/j2se/1.4.2/docs/api/java/util/Collections.html#rotate%28java.util.List,%20int%29) to each sub-array.

{{< highlight cfm >}}
<cfloop from="1" to="#arrayLen(data)#" index="i">
	<cfset collection.rotate(data[i], -1) />
</cfloop>
{{< / highlight >}}

Super cool eh? Here is the code sample in full. Happy coding, and thanks
for the interesting blog post Ray!

{{< highlight cfm >}}
<cffunction name="render" returntype="void" access="public" output="true">
	<cfargument name="arr" type="array" required="true" />

	<cfset var i = 0 />
	<cfset var x = 0 />

	<cfloop array="#arguments.arr#" index="i">
		<cfloop array="#i#" index="x">
			<cfset writeOutput("#x# ") />
		</cfloop>
	</cfloop>

	<cfset writeOutput("

	") />
</cffunction>


<cfset collection = createObject("java", "java.util.Collections") />
<cfset data = [
	["a", "b", "c"],
	["d", "e"]
] />

<!---
	Display the original data array
--->
<cfset render(data) />

<!---
	Rotate the arrays
--->
<cfloop from="1" to="#arrayLen(data)#" index="i">
	<cfset collection.rotate(data[i], -1) />
</cfloop>

<cfset render(data) />


<!---
	Rotate the arrays... again
--->
<cfloop from="1" to="#arrayLen(data)#" index="i">
	<cfset collection.rotate(data[i], -1) />
</cfloop>

<cfset render(data) />
{{< / highlight >}}