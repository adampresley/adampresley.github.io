---
author: "Adam Presley"
date: 2011-02-15T08:53:00Z
tags: ["development", "c++"]
title: "Asking For a List of Inputs on the Console in C/C++"
slug: "asking-for-a-list-of-inputs-on-the-console-in-cc"
---

Here's a question I got today:

> I'm trying to prompt the user to enter an array of strings. How do I
> do that?

Considering I haven't written a complete C/C++ application in over 5
years I admit I had to dust off the Google a bit, but luckily most of it
came back to me. To do this (using C, not C++) we'll need a pointer
array, each item eventually containing a string. This means we'll have
to use **malloc()** to allocate the memory to hold this information. We
then can use an old fashioned loop and **scanf()** to ask for the user
input, and stuff the result into our new array. For good measure we then
loop again and display what the user entered. Here's the code.

{{< highlight c >}}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {
	char** names[4];
	int i = 0;

	printf("Type in 4 names.");

	for (i = 0; i < 4; i++) {
		/*
		 * Use malloc for C apps. For C++,
		 * names[i] = new char*[128];
		 */
		names[i] = (char**) malloc(128);
		scanf("%s", (char*) names[i]);
	}

	printf("Now showing what you typed...");
	for (i = 0; i < 4; i++) {
		printf("%s", names[i]);
	}

	return 0;
}
{{< / highlight >}}

I admit it was a little fun dusting off that old knowledge, but it also
reminds me why I love languages like [ColdFusion](http://www.adobe.com/products/coldfusion/)
and [Groovy](http://groovy.codehaus.org/) so much. A lot of that memory management
nonsense is handled for you, which does make my life easier!
