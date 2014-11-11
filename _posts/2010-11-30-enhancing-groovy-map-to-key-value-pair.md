---
layout: post
title: Enhancing Groovy - Map to Key-Value Pair
date: 2010-11-30 11:26:00
author: Adam Presley
status: Published
tags: development groovy
slug: enhancing-groovy-map-to-key-value-pair
---

Here is another example of using Groovy's metaClass abilities to extend
the language. In this scenario I have a need to convert a standard
**Map** to a key-value pair, such as what would be used in an HTTP
POST. Continuing from my previous post about extending **String** to
have the ability to URL encode itself, we will now extend **Map** to
transform itself to a key-value pair. Paste this in your Groovy Console
and give it a try.  
  
**Edit:** Thanks to Tim Yates for bringing up the *join()*
command. Made it a lot cleaner!
  
{% highlight groovy %}
String.metaClass.encodeURL = {
	java.net.URLEncoder.encode(delegate)
}

Map.metaClass.toKeyValuePair = { 
	delegate.collect { key, value -> 
		"${key.encodeURL()}=${value.toString().encodeURL()}"
	}.join("&")
}

def sourceMap = [
	a: "123",
	b: "This is a test",
	c: "I_have 20% coverage of this & I want *lots* more.",
	d: "Whatever!"
]

sourceMap.toKeyValuePair()
{% endhighlight %}
  
And with that simple extension we can now easily convert a **Map**
to a key-value pair string. The astute among you may have noticed that
the value being added to the result string is using **toString()**,
and that is the value of the key is not a string you may get an object
address, or other such wonderfulness. True enough this method is
limited, and could probably be enhanced to be more sophisticated, but
that is for another time. Happy coding!
