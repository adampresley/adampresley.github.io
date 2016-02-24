---
author: "Adam Presley"
date: 2010-10-06T03:51:00Z
tags: ["development", "groovy"]
title: "Using Closures to Time Sections of Code in Groovy"
slug: "using-closures-to-time-sections-of-code-in-groovy"
---

Last night I was working on an import script in Groovy to bring data
back from a web service and into a MongoDB database. There's a lot of
data, and I wanted to know how much time the whole process was taking,
as well as how much time certain pieces of the script were taking. As
such I built a little closure that I could wrap around each code block I
wished to time and it would print to the console how many milliseconds,
seconds, or minutes that block took to execute. Here's a sample of code
demonstrating this.

{{< highlight groovy >}}
package com.adampresley.timing

class Main {
	static main(args) {
		/*
		 * Closure for timing stuff
		 */
		def timeMe = { Closure codeBlock ->
			def start = new Date().getTime()
			codeBlock()
			def end = new Date().getTime()

			def totalTime = end - start
			def verbage = ((totalTime > 60000) ? "${(totalTime / 1000) / 60}m" : ((totalTime > 1000) ? "${totalTime / 1000}s" : "${totalTime}ms"))
			println "Executed in ${verbage}"
		}

		/*
		 * Time something simple, like inserting 1000 hash maps
		 * into an array.
		 */
		timeMe {
			def iterations = 1000

			print "Inserting ${iterations} hash maps into an array... "
			def result = []

			(1..1000).each {
				result << [ firstName: "Adam", lastName: "Presley", age: 32 ]
			}
		}

		/*
		 * Do something bigger, like recurse over a directory
		 * structure and put all file names that start with
		 * 'A' into an array.
		 */
		timeMe {
			print "Get all file names that start with 'A'... "
			def result = []

			new File("C:/").eachFileRecurse { currentFile ->
				if (!currentFile.isDirectory()) {
					if (currentFile.name =~ /(?i)^a.*/) result << currentFile.path
				}
			}

			print "${result.size()} match(es) found. "
		}
	}
}
{{< / highlight >}}

As you can see I wrap each code block I want to time with the
**timeMe()** closure. In this example I provide two tests. The first
test is quick so I can show that the closure displays the time executed
in milliseconds. The second test is slightly longer so I can show the
closure is still smart enough to show the execution time in minutes.

And there you have it. That's some groovy coding yo! Happy coding!
