---
author: "Adam Presley"
date: 2011-10-28T10:55:00Z
tags: ["development", "python"]
title: "Starting to learn Python"
slug: "starting-to-learn-python"
---

So I am starting to learn Python. The catalyst for this was the Sublime
Text 2 editor, which is Python based, and has full plugin support
written in Python. So as such I set out to learn more about this
language. Initial impression is good. Since I am already fairly
particular about whitespace, I actually enjoy the fact it carries
meaning in this language.

One of the things I've learned more of today is list comprehension. Man
is that stuff cool. As an example of what I mean, imagine we have a list
of products in a catalog, each being a dictionary of name and price.
Then imagine that we have a second list of which of those products are
discounted, as well as a discount percent of 20%. Given a cart of three
items, one discounted, the other two not, I would like to calculate a
list of what my prices are for each item in the cart. Here is a code
snippet that does just that.

{{< highlight python >}}
#
# Craft a list of product names and their prices
#
products = [
	{ "name": "Baseball Cap", "price": 12.99 },
	{ "name": "T-Shirt", "price": 18.99 },
	{ "name": "Big Foam Finger", "price": 8.50 },
	{ "name": "Frosty Glass Mug", "price": 15.89 },
	{ "name": "Authentic Jersey", "price": 129.00 }
]

#
# This is a list of the current products on discount.
# Also determine what percentage discount.
#
discountedProducts = [ products[0], products[3] ]
discountPercent = 20.0

#
# Here be our shoppy cart.
#
cart = [ products[1], products[3], products[2] ]

#
# What are the new prices of my items in the cart?
#
prices = [ (item["price"] - (item["price"] * (discountPercent / 100))) if (item in discountedProducts) else item["price"] for item in cart ]
print "CART: %s" % (cart)
print "\nPRICES: %s" % (prices)
{{< / highlight >}}

The tricky line is line 30. The outer bracket indicates we are going to
return a list to a variable named prices. The is followed by a
calculation to determine the new price of an item based on the
discountPercent variable. The next bit of code is an **if** statement,
indicating that we are starting a ternary operator, the condition being
if the current item is in the discountedProducts list. If it is not the
default price is returned. The **for** statement indicates that we are
going to loop over our cart list. This is the nature of the list
comprehension. Act on a series of items, doing something to each. I
still have a way to go in understanding this really neat feature, and I
look forward to it! Happy coding!
