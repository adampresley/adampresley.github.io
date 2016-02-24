---
author: "Adam Presley"
date: 2010-08-25T12:37:00Z
tags: ["development", "coldfusion", "regex"]
title: "Adding Namespaces to HTML Tag Using Regular Expressions"
slug: "adding-namespaces-to-html-tag-using-regular-expressions"
---

My friend, [Mr. Steve Good](http://stevegood.org), approached me about a problem he was
having where he needed to add a namespace to an HTML tag at runtime.
Essentially he was parsing some dynamic HTML and wished to inject an
additional namespace into the HTML declaration. Regular expressions to
the rescue! The first test I ran was with a blank <HTML> tag. For this
example I will be injecting the SVG, or Scalable Vector Graphics
namespace. To accomplish this in ColdFusion we simply are using the
**replaceAll** method against the **String** class in Java.

{{< highlight cfm >}}
<cfset old = "<html>" />
<cfset new = old.replaceAll("(?i)(?<=<html)(.*?)>", "$1 xmlns:svg=""http://www.w3.org/2000/svg"">") />
<cfdump var="#new#" />
{{< / highlight >}}

The next task was to ensure that if the HTML tag already had a namespace
our regex won't erase it, but will instead append the new namespace to
it. Let's see that code.

{{< highlight cfm >}}
<cfset old = "<html xmlns=""http://www.w3.org/1999/xhtml"">" />
<cfset new = old.replaceAll("(?i)(?<=<html)(.*?)>", "$1 xmlns:svg=""http://www.w3.org/2000/svg"">") />
<cfdump var="#new#" />
{{< / highlight >}}

I love me some regular expressions. Happy coding!
