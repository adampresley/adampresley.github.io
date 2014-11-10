---
layout: post
title: Simple LRU Cache Class in PHP
date: 2009-05-27 12:58:00
author: Adam Presley
status: Published
tags: development php
slug: simple-lru-cache-class-in-php
---

In my ongoing effort to further enhance my PHP framework I started
recently working on a caching mechanism. For the time being I am using
the PEAR library's [Cache_Lite](http://pear.php.net/package/Cache_Lite)
(I've blogged on this before, so [check it out](#post/2007/05/caching-in-php-cache_lite))
to handle the details of persisting the cache data,
locking, and all that fun jazz. Cache_Lite, however, is a pretty simple
library and does not implement any particular algorithm regarding how
much cache to keep, or what stays in cache and what goes. It offers an
expiration mechanism, but that's about it. So with that in mind I set off
to learn about various caching and paging algorithms. They range, I
found out, from simple to complex.

In this blog entry I will demonstrate the creation of a very simple LRU,
or Least Recently Used, caching algorithm. I will provide downloads for
PHP source code at the end of this entry.

First let's talk about how the algorithm works. The concept behind LRU
is that you have a finite set of *slots* in some container. Our
container will be an array for simplicity. The LRU algorithm works off
the premise that you keep only the most recently used items in your
container, and those that get accessed are moved to the top of the
array. Adding a new item to the container will be put in at the top of
the array, and all subsequent items are shifted down. Any item at the
bottom of the array is dropped off from the container.

Items in the LRU array are going to be stored as a structure (array in
PHP) consisting of a *key* and *element*, which we will call a *node*.
The *element* is simply whatever data is to be stored. The *key* is the
identifier for the data element, and how we are able to insert and
retrieve elements, or items, from the array.

There are two primary operations to supprt this: **get** and **put**.
Let's start with **put** as it is the simplest in concept. The first
thing the **put** operation must do is *evict* any element at the bottom
of the array, then shift all of the elements above down one notch. So,
for example, of you have an array of 5 slots, and you are adding a new
element, you will remove the last item in the array (slot 4), then shift
items in slots 0 through 3 down into slots 1 through 4. From here you
then insert the new item into slot 0. Let's take a look at how that
works.

	:::php
	<?php

	public function put($key, $element) {
		$this->__evict();
		$this->__container[0] = $this->__createNode($key, $element);
	}

	private function __evict() {
		$index = 0;
		$newPosition = 0;

		array_pop($this->__container); // Drop off the element in the last slot.
		array_unshift($this->__container, ""); // Shift every element down one, and insert blank at the top.
	}

	private function __createNode($key, $element) {
		return array(
			"key" => $key,
			"element" => $element
		);
	}

	?>

Ok, that was easy enough. Now let's talk about the **get** operation.
**Get** starts with taking a single argument called *key*. It then
attempts to find the correct node by looping over the container array,
retrieving a node each iteration, then comparing the node's key to the
argument key. If we have a match we want to MOVE this node to the
**top** of the array since it is the most recently used node. Once this
is done we return the node. Here's what that looks like.

	:::php
	<?php

	public function get($key) {
		$index = 0;
		$node = "";

		for ($index = 0; $index < $this->__maxElements; $index++) {
			$node = $this->__container[$index];

			if (is_array($node)) {
				if ($node["key"] === $key) {
					$this->__moveToHead($index);
					$this->__stats["cacheHits"]++;
					return $node["element"];
				}
			}
		}

		$this->__stats["cacheMisses"]++;
		return false;
	}

	private function __moveToHead($index) {
		if ($index == 0) return;

		$element = array_splice($this->__container, $index, 1);
		array_unshift($this->__container, $element[0]);
	}

	?>

Using this class is pretty easy, so here is a simple test that provides
a series of data items, and a comma-delimited pattern of indexes to
those data items. The code then iterates over this list, simulating some
part of a program that is being asked for some data, and it attempts to
get the data from the cache. If it does not have it the code then puts
the data into the cache.

	:::php
	<?php

	require_once("LRUCache.php");

	$lru = new LRUCache(5);

	$db = array(
		"This is item 0",
		"This is item 1",
		"This is item 2",
		"This is item 3",
		"This is item 4",
		"This is item 5"
	);

	$testPattern = "0,2,4,1,0,5,3,1,5,3,0,1,5,2,5,1,4";

	// Implement test pattern.
	foreach (explode(",", $testPattern) as $index) {
		$item = $lru->get($index);

		if ($item !== false) {
			printf("Cache hit. - %s\n", $item);
		} else {
			printf("Cache miss for key %s. Adding %s to cache.\n", $index, $db[$index]);
			$lru->put($index, $db[$index]);
		}
	}

	printf("");
	$stats = $lru->getCacheStats();

	printf("Hits: %s", $stats["cacheHits"]);
	printf("Misses: %s", $stats["cacheMisses"]);

	?>

Go ahead, mess with the test pattern and see how many cache hits vs
cache misses there are. Kinda neat.

Happy coding!
