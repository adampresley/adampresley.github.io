---
layout: post
title: Convert Array of Strings to Integers Using jQuery
date: 2011-08-10 09:35:00
author: Adam Presley
status: Published
tags: development jquery javascript
slug: convert-array-of-strings-to-integers-using-jquery
---

Here's a small problem I was asked about the other day. How, using
jQuery, can you convert an array of number strings into an array of
integers? Using jQuery this is actually pretty simple. With an input
like this:  

	:::javascript
	["1", "2", "3"]

We want the end result to look like this:   

	:::javascript
	[1, 2, 3]

jQuery has a nifty method called *map()*that we can use to iterate over
an array operating on each item individually. The return values from the
*map()*method are collected back up into an array and returned. This
basically allows you to do something to each item in the array, and get
a new version. The code looks like this.  

	:::javascript
	/*
	 * Array of number strings.
	 */
	var input = ["1", "2", "3"];

	/*
	 * Convert this array of number strings to an array
	 * of integers. Use the jQuery map() function to
	 * operate against each item in the origin array,
	 * the end result being an array of transformed items.
	 */
	var output = $.map(input, function(item, index) {
		return window.parseInt(item);
	});

	/*
	 * Show our before and after on the console. (FF &amp; Chrome)
	 */
	console.log("Input: %o", input);
	console.log("Output: %o", output);

Go ahead and take this code for a spin in Firefox or Chrome and see the
magic of jQuery. Happy coding!
