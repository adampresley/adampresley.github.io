---
author: "Adam Presley"
date: 2011-09-01T13:40:00Z
tags: ["development", "javascript"]
title: "'Sorting and Removing Duplicates in JavaScript Arrays: Part 2'"
slug: "sorting-and-removing-duplicates-in-javascript-arrays-part-2"
---

Ok, so I said I'd do a follow up post, so here it is. I did some more
playing around with the removal of duplicates. I figured since I was
bothering to sort the array before removing duplicates I could improve
the algorithm a bit, and I was right. In fact, I uncovered a bug in the
first version of the algorithm, so this new one works better. How much?
Here is a screenshot of both methods (version 1 and 2) running against
the same set of arrays, and it shows how many iterations are run for
each array and algorithm.

![Dedupe Screenshot](http://s3.amazonaws.com/www.adampresley.com/posts/sorting-removing-dups-javascript-2.jpg)

Notice that in v1 of the algorithm, the first unit test has a duplicate
in it. But more importantly notice the large drop in iterations. The new
function looks like this. Happy coding!

{{< highlight javascript >}}
/**
 * Removes duplicates from an array of items. Does not return any
 * value as it removes duplicates from the original array itself. Use
 * of a comparator allows for comparing complex items. For example:
 *
 * <code>
 * var a = [ 2, 1, 1, 5 ];
 * a.dedupe({
 *    sortComparator: function(a, b) {
 *       if (a > b)
 *          return 1;
 *       else if (a < b)
 *          return -1;
 *       else
 *          return 0;
 *   },
 *   dedupeComparator: function(a, b) {
 *      return a === b;
 *   }
 * });
 * </code>
 *
 * @author Adam Presley
 *
 * @param comparators An object containing two keys: sortComparator and
 *  dedupeComparator. Both are functions taking two arguments. See
 *  the sort method for information on the sortComparator. The
 *  dedupeComparator takes two arguments, and expects a true/false
 *  return on if the items are the same or not.
 */
 Array.prototype.dedupe = function(comparators) {
 	this.sort(comparators.sortComparator);

 	var length = this.length, index = 0;
 	var spliceStart = 0, spliceLen = 0;
 	var item = 0, compareResult = 0, temp = 0;

 	if (length > 1) {
 		while (index < length - 1) {
 			item = this[index];

 			if (index + 1 < length) {
 				spliceStart = index + 1;
 				spliceLen = 0;
 				temp = this[spliceStart];

 				while ((compareResult = comparators.dedupeComparator.call(this, item, temp))) {
 					spliceLen++;

 					if (spliceStart + spliceLen >= length - 1) break;
 					temp = this[spliceStart + spliceLen];
 				}

 				if (spliceLen > 0) {
 					this.splice(spliceStart, spliceLen);
 					length -= spliceLen;
 				}
 			}

 			index++;
 		}
 	}
 };
{{< / highlight >}}
