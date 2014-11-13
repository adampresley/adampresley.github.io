---
layout: post
title: Enhancing Groovy - String to File
date: 2009-02-09 05:44:00
author: Adam Presley
status: Published
tags: development groovy
slug: enhancing-groovy-string-to-file
---
Although I'm sure this won't be **super** useful, I did find a use case
for it yesterday while writing a command line log parsing tool in
Groovy. One of the options I was providing users of this tool (mainly
me) is the ability to pass in a comma-delimited list of files to
process. Because of this I get a String that I can use *split()*
against, and have a Collection to work with. I also provided a way to
parse files that match a regular expression, so basically there are two
ways to specify files.

Because of this the parser class is expecting a Collection of **File**
objects. So then I was on task to find the best, or maybe just coolest,
way to convert that Collection of Strings into a Collection of File
objects.

For the first pass at this I use a simple **each** iteration like so:

{% highlight groovy %}
def pretendCommandLineArg = "/home/psykoprogrammer/Documents/test.xls,/home/psykoprogrammer/Documents/ClearConnectUserGuide.docx"
def fileList = []

pretendCommandLineArg.split(",").each {
	fileList << new File(it)
}

fileList.each {
	println "File Name: ${it.getName()}, Path: ${it.getCanonicalPath()}"
}
{% endhighlight %}

And that worked just fine. Then it occured to me that I could add a
metaClass enhancement to my already growing list of metaClass
enhancements that can convert a string to a File object. That's simple
enough to do.

{% highlight groovy %}
String.metaClass.toFile { basePath = "" ->
	new File("${basePath}${delegate}")
}
{% endhighlight %}

With that enhancement to the String class I can now do something like
this:

{% highlight groovy %}
"/home/adampresley/filea.txt".toFile()
"filea.txt".toFile("/home/adampresley/")
{% endhighlight %}

That's pretty neat, but what about the fact I have a command line
argument with a list of files delimited by a comma? Simple.

{% highlight groovy %}
def pretendCommandLineArg = "/home/adampresley/test.xls,/home/adampresley/testfile.txt"
def fileList = pretendCommandLineArg.split(",")*.toFile()

fileList.each {
	println "File Name: ${it.getName()}, Path: ${it.getCanonicalPath()}"
}
{% endhighlight %}

By using the spread operator I can apply my new **toFile()** String
method to each String in the Collection! Pretty cool eh? Although not
the most useful thing hopefully this gets your brain pumping on the
coolness that is Groovy! Happy coding!
