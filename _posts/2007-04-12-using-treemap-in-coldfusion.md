---
layout: post
title: Using TreeMap in ColdFusion
date: 2007-04-12 11:40:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: using-treemap-in-coldfusion
---
Have you ever found the need to maintain a structured list of items? Of
course! Have you ever needed to make sure those items stay in natural
order? Maybe. If you answered yes then you need the Java TreeMap class.

Let us say we are going to store a structure of employee names and their
titles associated to them. In this example we will keep just such a
list, insert them in random order, and you will see the output result
comes out in alphabetical (natural) order. The order is maintained based
on the KEY, by the way. So, without further ado, here is an example.

{% highlight cfm %}
<cfscript>
	treeMap = createObject("java", "java.util.TreeMap").init();

	writeOutput("Inserting the following employees in this order: Donny, Adam, Kujo, Johnny.");

	treeMap.put("Donny", "President");
	treeMap.put("Adam", "Engineer");
	treeMap.put("Kujo", "Director");
	treeMap.put("Johnny", "Support");

	writeOutput("Now show the contents of this tree in key order.");

	iter = treeMap.keySet().iterator();
	while (iter.hasNext()) {
		key = iter.next();
		writeOutput("Key: #key# Value: #treeMap.get(key)#");
	}
</cfscript>
{% endhighlight %}

In this example we instantiate the TreeMap class first. We then insert
the employees Donny, Adam, Kujo, and Johnny, making the value of these
keys their job titles. We then get an iterator for the keyset (an array
of the TreeMap keys) and loop while we still have keys, displaying the
key name, and its value. Here is what the output looks like.

	> Inserting the following employees in this order: Donny, Adam, Kujo, Johnny.
	>
	> Now show the contents of this tree in key order.
	> Key: Adam Value: Engineer
	> Key: Donny Value: President
	> Key: Johnny Value: Support
	> Key: Kujo Value: Director
