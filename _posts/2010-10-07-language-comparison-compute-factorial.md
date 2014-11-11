---
layout: post
title: "Language Comparison: Compute Factorial"
date: 2010-10-07 00:43:00
author: Adam Presley
status: Published
tags: development groovy coldfusion java
slug: language-comparison-compute-factorial
---

A friend of mine is in school and taking some classes in Java, and also
happens to be learning ColdFusion on the side. He was talking about the
class and how they were learning about recursion and how they used the
example of computing the factorial of a given number. Just for fun I
took that opportunity to do this in a couple of languages... just to see
what they look like side by side.

Please note there are no advantages or disadvantages to any of these
code samples, but instead is shown as something educational... food for
the brain if you will. :)

**ColdFusion**

{% highlight cfm %}
<cffunction name="findFactorialCF" returntype="numeric" access="public" output="false">
	<cfargument name="n" type="numeric" required="true" />
	<cfreturn (arguments.n EQ 1) ? 1 : arguments.n * findFactorialCF(arguments.n - 1) />
</cffunction>

<cfoutput>
#findFactorialCF(11)#
</cfoutput>
{% endhighlight %}

**Java**

{% highlight java %}
package com.adampresley.factorial

class Factorial {
	public static void main(String[] args) {
		System.out.println(findFactorialJava(11));
	}

	public static int findFactorialJava(int n) {
		return (n == 1) ? 1 : n * findFactorialJava(n - 1);
	}
}
{% endhighlight %}

**Groovy**

{% highlight groovy %}
def findFactorialGroovy = { n ->
	(n == 1) ? 1 : n * call(n - 1)
}

println findFactorialGroovy(11)
{% endhighlight %}

I love languages. Happy coding!
