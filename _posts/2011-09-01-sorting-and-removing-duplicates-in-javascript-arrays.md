---
layout: post
title: "Sorting and Removing Duplicates in JavaScript Arrays"
date: 2011-09-01 11:16:00
author: Adam Presley
status: Published
tags: development javascript
slug: sorting-and-removing-duplicates-in-javascript-arrays
---

Today I had a need to sort and remove duplicate entries from an array in
JavaScript. I figured I'd share the code for those interested. Here is
the code for sorting. This uses a simple Bubble Sort.

	:::javascript
	Array.prototype.sort = function(comparator) {
		var top = this.length;
		var newTop = 0, index = 0, temp = 0;

		do {
			newTop = 0;

			for (index = 1; index < top; index++) {
				if ((comparator.call(this, this[index - 1], this[index])) > 0) {
					temp = this[index];
					this[index] = this[index - 1];
					this[index - 1] = temp;

					newTop = index;
				}
			}

			top = newTop;
		} while(top > 0);
	};

For the task of removing duplicates I am simply looping and splicing.
I'm betting there is a more efficient way to do this, especially since I
am sorting the array before removing duplicates, but this will have to
do for now. I may do a follow up post with that in mind...

	:::javascript
	Array.prototype.dedupe = function(comparators) {
		this.sort(comparators.sortComparator);

		var index = 0, length = this.length, spliceIndex = 0, inner = 0;
		var item = 0, compareResult = 0;

		for (index = 0; index < length; index++) {
			item = this[index];
			if (index < length - 1 && length > index + 1) {
				for (inner = index + 1; inner < length; inner++) {
					compareResult = comparators.dedupeComparator.call(this, item, this[inner]);

					if (compareResult === true) {
						this.splice(index + 1, 1);
						length--;
					}
				}
			}
		}
	};

And finally an example of their use. Run it, and give it a try. Happy
coding!

	:::javascript
	var a = [
		1, 4, 3, 0, 4, 5, 1, 0, 7, 7, 2, 3, 10
	];

	a.dedupe({
		sortComparator: function(a, b) {
			if (a > b)
				return 1;
			else if (a < b)
				return -1;
			else
				return 0;
		},

		dedupeComparator: function(a, b) {
			return a === b;
		}
	});

	console.log("was [1, 4, 3, 0, 4, 5, 1, 0, 7, 7, 2, 3, 10], now %o", a);
