---
author: "Adam Presley"
date: 2009-03-19T08:08:00Z
tags: ["development", "coldfusion"]
title: "ColdFusion Random ID Generation"
slug: "coldfusion-random-id-generation"
---

Had a situation at work today where I needed a quasi-random ID to store
as a "key" in the database. This key would be handed to users for their
use over a given session for webservices.

At first I started with a UUID using the following code, but it occured
to me that for my audience that string was a bit long, so I wanted
something a little smaller.

{{< highlight cfm >}}
<cfset key = createObject("java", "java.util.UUID").randomUUID().toString() />
{{< / highlight >}}

The objective here is to create a function that we can use to generate a
fixed-length psudo-random set of digits to use a key to give to users.
We will make that length definable for better flexibility.

The first order of business is to define what characters will make up
our random key. For this function we are creating we will use the
alphabet and numbers.

{{< highlight cfm >}}
<cfset var chars = "abcdefghijklmnopqrstuvwxyz1234567890" />
{{< / highlight >}}

From here we will initialize Java's Random class to generate random
numbers for us. We will use this while creating our digits to determine
the index into the chars string to pull characters from. We will also
initialize a StringBuffer for faster string creation. Once we have what
we need, we will loop and append a series of random digits until we have
reached the specified length. Let's take a look.

{{< highlight cfm >}}
<cfcomponent name="RandomId">

	<cffunction name="generate" returntype="string" access="public" output="false">
		<cfargument name="numCharacters" type="numeric" required="false" default="8" />

		<cfset var chars = "abcdefghijklmnopqrstuvwxyz1234567890" />
		<cfset var random = createObject("java", "java.util.Random").init() />
		<cfset var result = createObject("java", "java.lang.StringBuffer").init(javaCast("int", arguments.numCharacters)) />
		<cfset var index = 0 />

		<cfloop from="1" to="#arguments.numCharacters#" index="index">
		<cfset result.append(chars.charAt(random.nextInt(chars.length()))) />
		</cfloop>

		<cfreturn result.toString() />
	</cffunction>

</cfcomponent>
{{< / highlight >}}

The usage of this component is similar to the method above:

{{< highlight cfm >}}
<cfset key = createObject("component", "RandomId").generate() />
{{< / highlight >}}

Happy coding!
