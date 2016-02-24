---
author: "Adam Presley"
date: 2010-08-03T06:19:00Z
tags: ["development", "coldfusion"]
title: "Merging Two ColdFusion Structs"
slug: "merging-two-coldfusion-structs"
---

Here's a quick little tidbit. My coworker [Steve Good](http://stevegood.org) asked me if I
knew of a quick way to merge two ColdFusion structures together, kind of
like how jQuery has the **$.extend()** method. Well there
**is** in fact a way to do this! And it's super easy.

Let's say you have structure one that has two keys, *firstName* and
*lastName*.

{{< highlight cfm >}}
<cfset struct1 = { firstName = "Adam", lastName = "Presley" } />
{{< / highlight >}}

And now we have structure two that has two keys, *firstName* and
*age*.

{{< highlight cfm >}}
<cfset struct2 = { firstName = "Michael", age = 33 } />
{{< / highlight >}}

Using a nifty ColdFusion method we can mash the two together in a single
line of code.

{{< highlight cfm >}}
<cfset structAppend(struct1, struct2) />
<!--- <cfset struct1.putAll(struct2) /> --->
{{< / highlight >}}

Woah, that was easy! The end result will be a structure that would look
like this.

{{< highlight cfm >}}
<cfset struct1 = { firstName = "Michael", lastName = "Presley", age = 33 } />
{{< / highlight >}}

Notice the commented out version. That is the underlying method to do
the same thing as StructAppend. The benefit? Nothing I can think of. :)

Enjoy, and happy coding!
