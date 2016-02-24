---
author: "Adam Presley"
date: 2010-09-03T11:37:00Z
tags: ["development", "groovy", "adam"]
title: "ADAM Script Lives!"
slug: "adam-script-lives"
---

It's been a **long** time since I last talked about, let alone
worked on ADAM script, or *A*dvanced *D*ata *A*cquisition and
*M*anipulation script. But last night I finally revived this project
that I haven't worked on since 2006. Yes, nearly 4 years have passed
since I last even **considered** my little data manipulation
language.

And now with [Groovy](http://groovy.codehaus.org/) being more powerful than ever, and me loving it
more than ever, I decided once again to revisit and rework this idea.
Groovy is a very powerful and versatile language on top of Java and the
JVM, and has language elements to actually support writing DSL's, or
Domain Specific Languages, which is what ADAM is. Consider the following
script.

	::groovy
	datasource ds1, {
		type "mysql"
		host "localhost"
		userName "user"
		password "password"
		catalog "test"
	}

This script demonstrates a sample of the language, and as you can see it
flows fairly easily. With a brief glance it should be apparent you are
setting up a data source. The neat thing about this script sample is
that it is technically Groovy code. Here's what it actually is.

	::groovy
	datasource("ds1", {
		type("mysql")
		host("localhost")
		userName("user")
		password("password")
		catalog("test")
	})

Since Groovy doesn't require parenthesis or semicolons we can build a
syntax that feels simpler and more natural. So the first argument is a
string. I'm able to lose the quotes due to the nifty missingProperty
method available, so when Groovy sees the word "ds1" without quotes, it
assumes it is a property. And when Groovy doesn't find the property in
the class it asks for missingProperty.

The second argument is a closure. Each line after that is technically a
method. The closure is executed in the context of the Datasource class
using the **with(closure)** syntax, which is uber-cool by the way.
This means that each method executed inside the closure will execute
within the context of the Datasource class.

So far Groovy DSLs are super cool and I think I may finally be on my way
to getting a working first-draft of my ADAM script language! Happy
coding!

