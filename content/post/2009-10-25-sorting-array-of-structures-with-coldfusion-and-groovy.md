---
author: "Adam Presley"
date: 2009-10-25T13:30:00Z
tags: ["development", "groovy", "coldfusion", "java"]
title: "Sorting Array of Structures with ColdFusion and Groovy"
slug: "sorting-array-of-structures-with-coldfusion-and-groovy"
---

<img src="http://s3.amazonaws.com/www.adampresley.com/posts/javabookcollection.jpeg" class="pull-right" style="margin-bottom: 10px; margin-left: 10px;" />
Today I had to work on a bit of code that dealt
with trying to tie in merchandising promotions against a shopping cart.
This code I'm working with is pretty old, so it's kind of like trying to
make a paper airplane out of the Constitution. :/ My boss is working on
one part of this, and I another, and we have decided we need to at least
restructure the cart, which is a simple array of structures, a little
bit so he can do what he needs with it. Long story short the changes we
needed required me to be able to sort the cart. Being the nerd that I am
I took a quick peek into the Java side of things, and found that the
Collections classes had something to offer.

The first item that I took note of was the [java.util.Comparator](http://java.sun.com/javase/6/docs/api/java/util/Comparator.html)
interface. After a little searching I came across a couple of code
samples that indicated how it worked. The interface allows you to create
your own class that implements a custom sorting mechanism that can be
used by the *sort()* method in the [java.util.Collection](http://java.sun.com/javase/6/docs/api/java/util/Collections.html)
class. This class you create must implement a single method, compare, that takes two
items as arguments. The arguments are two values that are being
compared, and you must return an integer of -1, 0, or 1 to indicate
"less than", "equal", or "greater than" to describe the relationship
between the two items. The type of the items being compared are defined
in the class implementation and arguments, as the Comparator interface
uses generics.

So in my example I wish to sort this array of products, each product
being a structure, by product ID first, and then if the product IDs
match, then by price. Here is what an example of the array would look
like.

{{< highlight cfm >}}
unsorted = [
	{
		productId = 3,
		title = "Widget ##3",
		price = 4.95,
		discounted = true
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
{{< / highlight >}}

Notice how our items are out of order (based on product ID and price),
and that we have two items with the productId of 3, but differing
prices. Let's look at how the comparator class would look.

{{< highlight java >}}
import java.util.*;

/*
 * Create a class that implements the java.util.Comparator interface.
 * This requires that we implement a single function named "compare"
 * that takes two items, and returns if the first if before, after,
 * or the same as the second item.
 *
 * In our case we will compare productId first. If they are the same
 * then we wanted to compare the prices.
 */
public class ProductComparator implements Comparator<coldfusion.runtime.Struct> {
	public int compare(coldfusion.runtime.Struct value1, coldfusion.runtime.Struct value2) {
		double price1 = value1.PRICE.toDouble();
		double price2 = value2.PRICE.toDouble();

		int productId1 = value1.PRODUCTID.toDouble();
		int productId2 = value2.PRODUCTID.toDouble();

		if (productId1 > productId2)
			return 1;
		else if (productId1 < productId2)
			return -1;
		else {
			if (price1 > price2)
				return 1;
			else if (price1 < price2)
				return -1;
			else
				return 0;
		}
	}
}
{{< / highlight >}}

The first thing of interest is the class definition. Notice how we
implement the Comparator interface. You'll also see those weird
brackets, followed by a reference to some type of ColdFusion class. This
is a Java generic. This states that the Comparator we are implementing
will have values in it of type "coldfusion.runtime.Struct". This class
is ColdFusion's structure implementation under the hood. Also notice our
compare method takes two arguments, both of type
**coldfusion.runtime.Struct**.

The method does a quick conversion of the values because I've found
ColdFusion has a bad habit of type-casting my values as strings. From
here we compare the two product IDs. If the first product ID is greater
than the second, we return 1. If it is less than the second we return
-1. If they are equal, however, we then want to do the same set of
comparisons to the price field. These return values allow the sorting
code to determine which out of two items should go after the other.

The use of this comparator requires that we get a Collections object and
call the sort method, passing in the array to sort, and the comparator.

{{< highlight cfm >}}
variables.collectionObject = createObject("java", "java.util.Collections");
variables.collectionObject.sort(someArray, variables.comparator);
{{< / highlight >}}

Let's take a look at the two pieces of code. The first is a CFC that
creates our sorter component. We will use Groovy to create the Java
class dynamically. The second is a test page that uses it. Happy
coding!!

{{< highlight cfm >}}
<cfcomponent>

	<cffunction name="init" returntype="productSorter" access="public" output="false">
		<cfscript>
			variables.collectionObject = createObject("java", "java.util.Collections");
			variables.comparator = __getComparator();

			return this;
		</cfscript>
	</cffunction>

	<cffunction name="Sort" returntype="array" access="public" output="false">
		<cfargument name="productArray" type="array" required="true" />

		<cfscript>
			var result = duplicate(arguments.productArray);

			variables.collectionObject.sort(result, variables.comparator);
			return result;
		</cfscript>
	</cffunction>

	<cffunction name="__getComparator" returntype="any" access="private" output="false">
		<cfimport prefix="g" taglib="/groovy" />

		<cfset arguments.object = "" />

		<cfoutput>
		<g:script args="#arguments#">

			import java.util.*;

			/*
			 * Create a class that implements the java.util.Comparator interface.
			 * This requires that we implement a single function named "compare"
			 * that takes two items, and returns if the first if before, after,
			 * or the same as the second item.
			 *
			 * In our case we will compare productId first. If they are the same
			 * then we wanted to compare the prices.
			 */
			public class ProductComparator implements Comparator<coldfusion.runtime.Struct> {
				public int compare(coldfusion.runtime.Struct value1, coldfusion.runtime.Struct value2) {
					double price1 = value1.PRICE.toDouble();
					double price2 = value2.PRICE.toDouble();

					int productId1 = value1.PRODUCTID.toDouble();
					int productId2 = value2.PRODUCTID.toDouble();

					if (productId1 > productId2)
						return 1;
					else if (productId1 < productId2)
						return -1;
					else {
						if (price1 > price2)
							return 1;
						else if (price1 < price2)
							return -1;
						else
							return 0;
					}
				}
			}

			arguments.object = new ProductComparator();

		</g:script>
		</cfoutput>

		<cfreturn arguments.object />
	</cffunction>

</cfcomponent>
{{< / highlight >}}


{{< highlight cfm >}}
<cfscript>

productSorter = createObject("component", "productSorter").init();

unsorted = [
	{
		productId = 3,
		title = "Widget ##3",
		price = 4.95,
		discounted = true
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

<p>
	This page demonstrates using Groovy to create a custom comparator used to sort
	an array of structures. In this example we will have an array of products. We
	wish to sort these products by productId first, and if the productId matches,
	we will then sort by price.
</p>

<fieldset style="padding: 10px; margin-bottom: 15px;">
	<legend>Unsorted Array of Products</legend>

	<cfdump var="#unsorted#" />
</fieldset>

<fieldset style="padding: 10px; margin-bottom: 15px;">
	<legend>Sorted Array of Products</legend>

	<cfdump var="#sorted#" />
</fieldset>
{{< / highlight >}}

**UPDATE** (11/13/2009):
See <#post/2009/11/sorting-array-of-structures-with-only-coldfusion/522a996b6851e13e760cd12b>
for a demonstration of sorting an array of structures without Groovy,
and in pure ColdFusion.
