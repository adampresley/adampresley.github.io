---
layout: post
title: Useful Date/Time Class for PHP
date: 2009-02-01 18:56:00
author: Adam Presley
status: Published
tags: development php
slug: useful-datetime-class-for-php
---

As programmers we should always strive to better ourselves by improving
our craft by taking pride that what we build is not only useful, but can
continue to be useful for a long time to come. Part of this strategy
involves writing code that exhibits resuse, high cohesion, and loose
coupling. Therefore every developer should, over time, develop, or at
least acquire, sets of tools and code to help achive this goal.

Concepts like frameworks and design patterns help us achieve this. I
have been working a few contract projects in [PHP](http://www.php.net) lately and have had
many opportunities to tweak, fine tune, and improve my home-brewed PHP
framework. One of these days I will share this work as a complete
framework for download and use, but for now it will simply have to
suffice to drop small pieces to the public.

One common task a developer will find themselves doing is manipulating
dates. This could be for the purposes of display, SQL queries, or a
plethora of other needs, but regardless of the reason having a useful
set of classes and functions for manipulating dates and times is handy.

In my framework I have built a simple class to do the most common date
tasks that I find myself repeating over and over. In my case I have
opted for a class that implements all static methods, so this is not
meant to be instantiated. And it is also only intended for PHP 5.2 and
higher.

Most the of the functions are pretty simple to follow, and there is
ample documentation attached. A couple of items worth noting are the
[date_parse](http://www.php.net/manual/en/function.date-parse.php)
and the [mktime](http://www.php.net/manual/en/function.mktime.php)
functions. In PHP, the [date](http://www.php.net/manual/en/function.date.php)
function is what allows us to format a date into whatever string we
desire to craft. The **mktime** function allows us to specify the values
for each part of a date and time string, such as the day, month, year,
hour, and so on.

What is usually tricky is making functions to format a date from various
sources. For example, a SQL date string usually looks like *2009-02-01*,
where a user in the United States might enter a value on a form like
*2/1/2009*. This is where the **date\_parse** function comes in handy.
It is smart enough to take these various types of input and split them
into their appropriate parts. The end result is an array of its parts.
For example.

	:::php
	$date = "02/01/2009";
	$split = date_parse($date);

	// $split["month"] = "02"
	// $split["day"] = "01"
	// $split["year"] = "2009"

As we can see the date has been split up into each of its parts. We can
then take these parts and pass them to **mktime** to build the value
necessay to pass to the **date** function for formatting.

I have linked to a download containing my date/time class, as well as
the documentation for it. Enjoy, and happy coding!

*Link no longer available*
