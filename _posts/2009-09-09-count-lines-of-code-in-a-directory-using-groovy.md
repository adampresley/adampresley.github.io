---
layout: post
title: Count Lines of Code in a Directory using Groovy
date: 2009-09-09 13:39:00
author: Adam Presley
status: Published
tags: development groovy
slug: count-lines-of-code-in-a-directory-using-groovy
---

So here's a tasty morsel. I wrote a quick little script to count the
number of lines of code in a ColdFusion project. Why? To make stuff look
impressive, or course! So here we make use of Groovy's very powerful
extensions to the Java File object, and its ability to easily iterate.
Take a look.  

{% highlight groovy %}
lineCount = 0;

new File("C:\\code\\someProject").eachDirRecurse {
	dir ->

	dir.eachFileMatch(~/.*\.(cfm|cfc|js)/) {
		file ->

		file.eachLine { lineCount++; }
	}
}

println "Line count = " + lineCount;
{% endhighlight %}

What's neat about this snippet is how beautifully simple it is. Right
out of the box we create a new File object against a directory, and
proceed to call the **eachDirRecurse** method. This method is a closure
that iterates over directory entries.   
  
Once in the closure we then execute the **eachFileMatch** method against
the directory (*dir* variable), passing it a regular expression that
states that we wish to get all files that match *.cfm, *.cfc, or
*.js. This in turn is ALSO a closure.

In this closure we then call the **eachLine** closure against the file
object (*file* variable), which iterates over all lines in a text file,
incrementing our *lineCount* variable for each iteration.  
  
It's so simple it's beautiful. I love me some Groovy!
