---
author: "Adam Presley"
date: 2009-10-06T10:54:00Z
tags: ["development", "groovy", "coldfusion"]
title: "Selective Merging of Structures using Regular Expressions in ColdFusion"
slug: "selective-merging-of-structures-using-regular-expressions-in-coldfusion"
---

Today I had a small situation where there is a structure in the session
scope that, when a form is submitted with address information, put this
address data into that session scoped structure. However, there is a
checkbox and a submit button on the form that get sent along with the
form scope. I did not want to do a straight structAppend and get those
values as well.

So I decided to build a function using the power and speed of Groovy to
merge two structures together, but only keys that match a regular
expression criteria. The use of such a function would look like this.

{{< highlight cfm >}}
<cfset session.checkOut = ReMergeStructuresByKey(session.checkOut, form, '(?i)billing[a-z0-9]+|shipping[a-z0-9]+') />
{{< / highlight >}}

In this example I want to merge keys from the form scope into the
session.checkOut structure where the key name starts with "billing" or
"shipping". The resulting structure in session.checkOut will have not
only its original members, but all the new address information as well.

Here is the function that achieves this. Note that I am using a modified
CFGroovy script as described [here](#post/2009/08/cfgroovy2-and-arguments-scope-in-functions).

{{< highlight cfm >}}
<cffunction name="ReMergeStructuresByKey" returntype="any" access="public" output="true">
	<cfargument name="source" type="struct" required="true" />
	<cfargument name="toMerge" type="struct" required="true" />
	<cfargument name="searchPattern" type="string" required="true" />
	<cfargument name="overwrite" type="boolean" required="false" default="true" />

	<cfimport prefix="g" taglib="groovyEngine" />
	<cfset arguments.result = duplicate(arguments.source) />
	<cfset arguments.overwrite = javaCast("boolean", arguments.overwrite) />

	<cfoutput>
	<g:script args="#arguments#">
		import java.util.regex.*

		keys = arguments.toMerge.keySet().iterator()
		key = ""
		index = 0
		exists = 0
		go = false

		Pattern p = Pattern.compile(arguments.searchPattern)
		Matcher m = null

		while (keys.hasNext()) {
			key = keys.next()
			if ((m = p.matcher(key)).matches()) {
				exists = arguments.source.containsKey(key)
				go = ((exists && arguments.overwrite) || !exists)
				if (go) arguments.result.put(key, arguments.toMerge.get(key))
			}
		}
	</g:script>
	</cfoutput>

	<cfreturn arguments.result />
</cffunction>
{{< / highlight >}}

Happy coding.
