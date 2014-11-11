---
layout: post
title: "Regex Merging of Structures in Groovy: Revisited"
date: 2011-01-06 21:03:00
author: Adam Presley
status: Published
tags: development groovy
slug: regex-merging-of-structures-in-groovy-revisited
---

Some time ago I blogged about using CFGroovy to build a function to take
two structures and merge their keys together selectively based on a
regular expression. Today I revisit that to make a smarter function, and
way more trim. The enhancement here is to allow the user to pass in an
array of structures, allowing for merging of as many structures as the
user needs. We also cleaned up the code a bit to be more ***Groovy***!

{% highlight groovy %}
def reMerge(structArray, regex) {
   def result = [:]
   def p = (regex instanceof java.util.regex.Pattern) ? regex : ~/${regex}/

   structArray.each { it.each { k, v -> if (p.matcher(k).matches()) result[k] = v } }
   result
}

def s1 = [ firstName: "Adam", companyName: "adampresley.com" ]
def s2 = [ age: 32 ]
def s3 = [ lastName: "Presley" ]

def list = [ s1, s2, s3 ]
def result = reMerge(list, ~/(?i).*name/)

result
{% endhighlight %}

Take that sample snippet and drop it in GroovyConsole and run it. As you
can see it will give you a new structure with only the keys from the
three source structures that have the word "name" in it. You may also
notice the reMerge function can take the regular expression as a string
**or** a Pattern object for convenience.  

Groovy is fun! Happy coding!
