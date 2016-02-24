---
author: "Adam Presley"
date: 2009-11-13T16:16:00Z
tags: ["development", "coldfusion"]
title: "Sorting Array of Structures with only ColdFusion"
slug: "sorting-array-of-structures-with-only-coldfusion"
---

In a previous post, [Sorting Array of Structures with ColdFusion and
Groovy](#post/2009/10/sorting-array-of-structures-with-coldfusion-and-groovy/522a996a6851e13e760cd042),
I talked about how one can use the Java Comparator interface
to build a class that, when used with the Java Collections class, can
sort an array of structures on multiple criteria. I demonstrated how to
first compare the product ID, then the price should the product IDs be
the same.

Just the other day I received a question on this post.

> Derek
> November 12th, 2009
>
> Is there a way to do this without Groovy?

The short answer? Of course! The Java Comparator class we created is
simply a small part of the Bubble Sort algorithm. The Bubble Sort
algorithm is simple in premise. It repeatedly iterates over a list of
items, comparing adjacent items, and if they are in the wrong order,
swaps them. Passes are continued until no more items need swapping. So
in this post we will look at how to do this in pure ColdFusion script
(yup, doing all CF9 in my development environment, and thus the
scripting, cause I love it!).

We are going to need a few variables to set up. First a copy of an array
passed in as an argument, as well as its size. We'll need two looping
variables, one for the outer passes, done over the whole array, and an
inner for comparing adjacent items. We will also need a temporary
holding structure for copying items when they need swapping. Since we
are comparing product IDs and prices, we'll need variables for these as
well.

{{< highlight cfm >}}
var result = duplicate(arguments.ary);
var size = arrayLen(result);
var pass = 0; var i = 0;
var temp = {};

var price1 = 0.00; var price2 = 0.00;
var productId1 = 0; var productId2 = 0;
{{< / highlight >}}

To start we'll need two loops. The first is the outer pass which will
iterate over all items in the array. The inner loop will go over each
one, comparing adjacent items. Then we'll get the product IDs and
prices.

{{< highlight cfm >}}
for (pass = 1; pass <= size; pass++) {
	for (i = 1; i <= size - pass; i++) {
		productId1 = result[i].productId;
		productId2 = result[i + 1].productId;

		price1 = result[i].price;
		price2 = result[i + 1].price;
	}
}
{{< / highlight >}}

As you can see we are getting product IDs for the current index, as well
as the next. We do the same for prices too. We will now compare the
values. If the first product ID is greater than the second, we need to
swap. If they are the same, we want to compare prices. If the first
price is greater than the second, we need to swap. Otherwise we just
leave the items where they are at.

{{< highlight cfm >}}
if (productId1 > productId2) {
	temp = duplicate(result[i]);
	result[i] = duplicate(result[i + 1]);
	result[i + 1] = temp;
} else if (productId1 == productId2) {
	if (price1 > price2) {
		temp = duplicate(result[i]);
		result[i] = duplicate(result[i + 1]);
		result[i + 1] = temp;
	}
}
{{< / highlight >}}

Once all is done, return the result. Below is the component in full (in
both CF9 and CF8), as well as a test page. Happy coding!!

**productSorter_cf9.cfc (ColdFusion 9):**

{{< highlight cfm >}}
component output="false" {
	public productSorter_cf9 function init() output="false" {
		return this;
	}

	public array function Sort(required array productArray) output="false" {
		return __sort(arguments.productArray);
	}

	private array function __sort(required array ary) output="false" {
		var result = duplicate(arguments.ary);
		var size = arrayLen(result);
		var pass = 0; var i = 0;
		var temp = {};

		var price1 = 0.00; var price2 = 0.00;
		var productId1 = 0; var productId2 = 0;

		/*
		 * Iterate over the entire array. This equates to how many
		 * comparisons we must make.
		 */
		for (pass = 1; pass <= size; pass++)  {
			/*
			 * Inner loop checks product at i against i + 1.
			 * If the product ID is higher, swap the items.
			 * If they are the same, check the price. Swap if higher.
			 */
			for (i = 1; i <= size - pass; i++) {
				productId1 = result[i].productId;
				productId2 = result[i + 1].productId;

				price1 = result[i].price;
				price2 = result[i + 1].price;

				if (productId1 > productId2) {
					temp = duplicate(result[i]);
					result[i] = duplicate(result[i + 1]);
					result[i + 1] = temp;
				} else if (productId1 == productId2) {
					if (price1 > price2) {
						temp = duplicate(result[i]);
						result[i] = duplicate(result[i + 1]);
						result[i + 1] = temp;
					}
				}
			}
		}

		return result;
	}
}
{{< / highlight >}}

**productSorter.cfc (ColdFusion 8):**

{{< highlight cfm >}}
<cfcomponent output="false">

	<cffunction access="public" name="init" output="false" returntype="productSorter">
		<cfreturn this />
	</cffunction>

	<cffunction access="public" name="Sort" output="false" returntype="array">
		<cfargument name="productArray" required="true" type="array" />

		<cfreturn __sort(arguments.productarray) />
	</cffunction>

	<cffunction access="private" name="__sort" output="false" returntype="array">
		<cfargument name="ary" required="true" type="array" />

		<cfscript>

			var result = duplicate(arguments.ary);
			var size = arrayLen(result);
			var pass = 0; var i = 0;
			var temp = {};

			var price1 = 0.00; var price2 = 0.00;
			var productId1 = 0; var productId2 = 0;

			/*
			 * Iterate over the entire array. This equates to how many
			 * comparisons we must make.
			 */
			for (pass = 1; pass <= size; pass++) {
				/*
				 * Inner loop checks product at i against i + 1.
				 * If the product ID is higher, swap the items.
				 * If they are the same, check the price. Swap if higher.
				 */
				for (i = 1; i <= size - pass; i++) {
					productId1 = result[i].productId;
					productId2 = result[i + 1].productId;

					price1 = result[i].price;
					price2 = result[i + 1].price;

					if (productId1 > productId2) {
						temp = duplicate(result[i]);
						result[i] = duplicate(result[i + 1]);
						result[i + 1] = temp;
					}
					else if (productId1 == productId2) {
						if (price1 > price2) {
							temp = duplicate(result[i]);
							result[i] = duplicate(result[i + 1]);
							result[i + 1] = temp;
						}
					}
				}
			}

			return result;
		</cfscript>
	</cffunction>
</cfcomponent>
{{< / highlight >}}

**Test Page:**

{{< highlight cfm >}}
<cfscript>

/*
 * Uncomment the next line for CF8, and comment out the line
 * following that.
 *
 * productSorter = createObject("component", "productSorter").init();
 */
productSorter = new productSorter_cf9();

unsorted = [
	{
		productId = 3,
		title = "Widget ##3",
		price = 6.95,
		discounted = true
	},
	{
		productId = 4,
		title = "Widget ##4",
		price = 6.75,
		discounted = false
	},
	{
		productId = 1,
		title = "Widget ##1",
		price = 10.95,
		discounted = false
	},
	{
		productId = 2,
		title = "Widget ##2",
		price = 11.95
	},
	{
		productId = 3,
		title = "Widget ##3",
		price = 5.95,
		discounted = false
	}
];

sorted = productSorter.Sort(unsorted);

</cfscript>
{{< / highlight >}}
