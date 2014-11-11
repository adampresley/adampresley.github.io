---
layout: post
title: Python itertools - Filtering items
date: 2013-02-12 13:32:00
author: Adam Presley
status: Published
tags: python itertools
slug: python-itertools-filtering-items
---

Continuing my exploration of the **itertools** module in Python I wanted
to look at the **ifilter()** method. **ifilter()** is a method that
returns an iterator of items that return true from a test function (or
lambda expression). In this sample I have an array of products. I want
to get a list of products that have inventory in stock and are allowed
to be drop-shipped.  
  
{% highlight python %}
import json
from itertools import ifilter

products = [
   {"id": "SHIRT-BLU15", "category": "T-Shirt", "color": "Blue", "size": "large", "instock": 125, "backorder": 50, "shipTypes": ["tostore", "dropship"], "price": 25.99},
   {"id": "SHIRT-BLU13", "category": "T-Shirt", "color": "Blue", "size": "small", "instock": 105, "backorder": 20, "shipTypes": ["tostore", "dropship"], "price": 23.99},
   {"id": "SHIRT-RED15", "category": "T-Shirt", "color": "Red", "size": "large", "instock": 145, "backorder": 20, "shipTypes": ["tostore"], "price": 26.99},
   {"id": "SHIRT-RED13", "category": "T-Shirt", "color": "Red", "size": "small", "instock": 25, "backorder": 75, "shipTypes": ["tostore"], "price": 20.99},
   {"id": "SHIRT-GRN15", "category": "T-Shirt", "color": "Green", "size": "large", "instock": 102, "backorder": 0, "shipTypes": ["tostore", "dropship"], "price": 21.99},
   {"id": "SHIRT-GRN13", "category": "T-Shirt", "color": "Green", "size": "small", "instock": 0, "backorder": 100, "shipTypes": ["tostore", "dropship"], "price": 21.99},
   {"id": "SHIRT-PUR15", "category": "T-Shirt", "color": "Purple", "size": "large", "instock": 25, "backorder": 60, "shipTypes": ["tostore"], "price": 27.99},
   {"id": "SHIRT-PUR13", "category": "T-Shirt", "color": "Purple", "size": "small", "instock": 0, "backorder": 100, "shipTypes": ["tostore"], "price": 26.99}
]

#
# Give me all shirts that are in stock and allow drop-ship
#
matches = [item for item in ifilter(lambda k: k["instock"] > 0 and "dropship" in k["shipTypes"], products)]

for i in matches:
   print "Item ID %s is in stock and available for drop-ship" % i["id"]
{% endhighlight %}

Much like most Python list and array manipulation methods I'm using a list comprehension here. The cool part is the call to **ifilter()**. The first argument is a lambda expression that checks the current item to see if we have a value greater than zero for *instock*, and if
*dropship* is an item in the *shipTypes* array. The end result is an array of items that match, and I loop over that to show what's
available.  
  
The more I use Python's list comprehensions and the **itertools** module the more I find I like it! Cheers, and happy coding!

