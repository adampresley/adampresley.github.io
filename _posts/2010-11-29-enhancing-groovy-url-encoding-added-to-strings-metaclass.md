---
layout: post
title: Enhancing Groovy - URL Encoding Added to String's Metaclass
date: 2010-11-29 22:25:00
author: Adam Presley
status: Published
tags: development groovy
slug: enhancing-groovy-url-encoding-added-to-strings-metaclass
---
I've talked before about enhancing the Groovy language through the
metaclass extension abilities. Here's a short post that adds a handy
snippet to the Groovy String class to URL encode strings. Normally this
is done through the **java.net.URLEncoder** class, but we are going
to add it to the string class so we can do something like this.  

{% highlight groovy %}
def someValue = "This is some value"
def someKey = "key"
def result = "${someKey.encodeURL()}=${someValue.encodeURL()}"
{% endhighlight %}

Seems like a pretty handy feature. And doing this is super easy.  

{% highlight groovy %}
String.metaClass.encodeURL = {
   java.net.URLEncoder.encode(delegate)
}
{% endhighlight %}

There are a couple of tricks behind this.   
  
1. The *metaClass* attribute in Groovy is what allows us to extend the language. Pimp.  
* The *encodeURL* is a new method, attached as a closure.  
* The **encode()** method does the real work. Notice no *return* statement. Groovy returns the last value in the method. That would be the encoded string in this case. 

And there you have it. Short, simple, and to the point. Happy coding!
