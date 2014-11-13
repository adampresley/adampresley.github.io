---
layout: post
title: Sorting a Complex Array of Products in PHP
date: 2009-11-17 04:39:00
author: Adam Presley
status: Published
tags: development php
slug: sorting-a-complex-array-of-products-in-php
---
I've blogged twice recently about sorting an array of structures in
ColdFusion, using both pure CF, and dipping into Groovy. This time we'll
look at how we can easily do the same task in PHP. In this post I will
take the same problem, a complex array containing a structure (in PHP,
an associative array), and sort it. The sort criteria in this case is
that if the product IDs are the same, we compare the prices of the
products and sort there. So we have a two-level sort: Product ID, then
price.

To do this in PHP the method **usort** will come in handy. The **usort**
method takes two parameters. The first parameter is the array to be
sorted (passed by reference, so make sure you keep a copy if you need
the original), and a function to compare two items.

The comparator function simply takes two items. It is up to us to
compare them how we like, and return zero (0) for equal, one (1) for the
first item being greater (or after) the second item, or negative one
(-1) for the first item coming before the second item, or less than.
Let's take a look at how that function would work.

{% highlight php %}
<?php

function comparator($a, $b) {
	$productId1 = $a["productId"];
	$productId2 = $b["productId"];

	$price1 = $a["price"];
	$price2 = $b["price"];

	if ($productId1 == $productId2) {
		if ($price1 == $price2)
			return 0;
		else
			return ($price1 > $price2) ? 1 : -1;
	} else {
		return ($productId1 > $productId2) ? 1 : -1;
	}
}

?>
{% endhighlight %}

As you can see, we compare item $a and item $b. If the the two product
IDs are the same, we'll compare prices. To use this is quite easy. You
pass an array into the **usort** method, and watch the magic happen.
This is how that works.

{% highlight php %}
<?php

function comparator($a, $b) {
	$productId1 = $a["productId"];
	$productId2 = $b["productId"];

	$price1 = $a["price"];
	$price2 = $b["price"];

	if ($productId1 == $productId2) {
		if ($price1 == $price2)
			return 0;
		else
			return ($price1 > $price2) ? 1 : -1;
	} else {
		return ($productId1 > $productId2) ? 1 : -1;
	}
}

$unsorted = array(
	array(
		"productId" => 3,
		"title" => "Widget #3",
		"price" => 6.95,
		"discounted" => true
	),
	array(
		"productId" => 4,
		"title" => "Widget #4",
		"price" => 6.75,
		"discounted" => false
	),
	array(
		"productId" => 1,
		"title" => "Widget #1",
		"price" => 10.95,
		"discounted" => false
	),
	array(
		"productId" => 2,
		"title" => "Widget #2",
		"price" => 11.95
	),
	array(
		"productId" => 3,
		"title" => "Widget #3",
		"price" => 5.95,
		"discounted" => false
	)
);

$sorted = $unsorted;
usort($sorted, comparator);

?>
{% endhighlight %}

Notice how we create a copy of *$unsorted* as a variable named
*$sorted* first. This is because the **usort** method works by
reference, and will modify the original array. So we need a copy first
to preserver the original (unless of course you don't care about the
original).

And that is all! Below is the code sample in full. Cheers, and happy
coding!

{% highlight php %}
<?php

function comparator($a, $b) {
	$productId1 = $a["productId"];
	$productId2 = $b["productId"];

	$price1 = $a["price"];
	$price2 = $b["price"];

	if ($productId1 == $productId2) {
		if ($price1 == $price2)
			return 0;
		else
			return ($price1 > $price2) ? 1 : -1;
	} else {
		return ($productId1 > $productId2) ? 1 : -1;
	}
}

$unsorted = array(
	array(
		"productId" => 3,
		"title" => "Widget #3",
		"price" => 6.95,
		"discounted" => true
	),
	array(
		"productId" => 4,
		"title" => "Widget #4",
		"price" => 6.75,
		"discounted" => false
	),
	array(
		"productId" => 1,
		"title" => "Widget #1",
		"price" => 10.95,
		"discounted" => false
	),
	array(
		"productId" => 2,
		"title" => "Widget #2",
		"price" => 11.95
	),
	array(
		"productId" => 3,
		"title" => "Widget #3",
		"price" => 5.95,
		"discounted" => false
	)
);

$sorted = $unsorted;
usort($sorted, comparator);

?>

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
	<title>Sorting</title>
</head>

<body>
	<h1>Sorting</h1>

	<strong>Unsorted:</strong>
	<ul>
		<?php

		foreach ($unsorted as $item) {
			printf("<li>%s (%s) - %s</li>\n",
				$item["title"],
				$item["productId"],
				$item["price"]);
		}

		?>
	</ul>

	<strong>Sorted:</strong>
	<ul>
		<?php

		foreach ($sorted as $item) {
			printf("<li>%s (%s) - %s</li>\n",
				$item["title"],
				$item["productId"],
				$item["price"]);
		}

		?>
	</ul>

</body>
</html>
{% endhighlight %}
