---
layout: post
title: JavaScript Functional Goodness Part 2
date: 2013-11-15 14:21:00
author: Adam Presley
status: Published
tags: development javascript
slug: javascript-functional-goodness-part-2
---

A couple of days ago I posted an entry on some basic functional-style
programming constructs that I've been *slowly* learning. I am back
to post part 2. Here's the refresher.

Here is the JSON data that I was working with. It was being used
to feed a line chart.

{% highlight javascript %}
{
   "seriesTotals": [
      3,
      77,
      54,
      65,
      64,
      56,
      41,
      14,
      63,
      99,
      63,
      30,
      41,
      24,
      12,
      42,
      53,
      51,
      50,
      35,
      27,
      22,
      58,
      65,
      40,
      51,
      44,
      27,
      12,
      66,
      64
   ],
   "labels":    [
      "Oct 13",
      "Oct 14",
      "Oct 15",
      "Oct 16",
      "Oct 17",
      "Oct 18",
      "Oct 19",
      "Oct 20",
      "Oct 21",
      "Oct 22",
      "Oct 23",
      "Oct 24",
      "Oct 25",
      "Oct 26",
      "Oct 27",
      "Oct 28",
      "Oct 29",
      "Oct 30",
      "Oct 31",
      "Nov 1",
      "Nov 2",
      "Nov 3",
      "Nov 4",
      "Nov 5",
      "Nov 6",
      "Nov 7",
      "Nov 8",
      "Nov 9",
      "Nov 10",
      "Nov 11",
      "Nov 12"
   ],
   "dayNames":    [
      "Sunday",
      "Monday",
      "Tuesday",
      "Wednesday",
      "Thursday",
      "Friday",
      "Saturday",
      "Sunday",
      "Monday",
      "Tuesday",
      "Wednesday",
      "Thursday",
      "Friday",
      "Saturday",
      "Sunday",
      "Monday",
      "Tuesday",
      "Wednesday",
      "Thursday",
      "Friday",
      "Saturday",
      "Sunday",
      "Monday",
      "Tuesday",
      "Wednesday",
      "Thursday",
      "Friday",
      "Saturday",
      "Sunday",
      "Monday",
      "Tuesday"
   ]
}
{% endhighlight %}

In the last post I wanted to sum up the series totals. In this post I want
to determine which day of the week has the highest number. Each number
in the series is associated with a day of the week, and I needed to
determine which day of the week has the highest summed value.

This can be done in a typical set of loops and sums like so.

{% highlight javascript %}
var totalsByDay = {};
var day = "";

for (var index = 0; index < data.seriesTotals.length; index++) {
    day = data.dayNames[index];
    if (!totalsByDay.hasOwnIndex(day)) totalsByDay[day] = 0;

    totalsByDay[day] += data.seriesTotals[index];
}

var maxDay = "";
var maxDayValue = 0;

for (var key in totalsByDay) {
    if (totalsByDay.hasOwnProperty(key)) {
        if (totalsByDay[key] > maxDayValue) {
            maxDayValue = totalsByDay[key];
            maxDay = key;
        }
    }
}

// maxDay == Tuesday
// maxDayValue = 335
{% endhighlight %}

Once again there is nothing specifically wrong with this code (as far as I know
as I didn't actually test it).
It first loops over the series total values and compiles an object where each key is
a day of the week with a value of the total number of items for that day. It then performs
another loop over this new structure and determines which item has the highest value.

To try a functional approach I decided to use map/reduce functions and a max function.
The map/reduce will create an array of object where the key is the day of the week
and the value is a number from the series totals. The reduce part of the function
will combine each day of the week object into a single key/value pair representing
the total items for each given day. In other words the map function will produce
something like this.

{% highlight javascript %}
[
    { key: "Sunday", value: 3 },
    { key: "Monday", value: 77 },

    ...

    { key: "Sunday", value: 14 }

    ...
]
{% endhighlight %}

And so on. The reduce function will sum up each day into a single object
for each day like so.

{% highlight javascript %}
{
    "Sunday": 60,
    "Monday": 229,

    ...
}
{% endhighlight %}

And so on. Here's what that code looks like looks like.

{% highlight javascript %}
/*
 * Sum up each day. The items array will be filled with objects that line
 * up items per day with the appropriate day of the week.
 */
var mapIdx = 0; // keep up with the day of the week
var totalsByDay = Util.reduce(
    {},
    Util.map(data.seriesTotals, function(item) { var r = { key: data.dayNames[mapIdx], value: item }; mapIdx++; return r; }),
    function(a, b) {
        if (!a.hasOwnProperty(b.key)) a[b.key] = 0;
        a[b.key] += b.value;
        return a;
    }
);
{% endhighlight %}

Much like last time the reduce function takes three arguments. The first is the starting value. In our
case that is a blank object. The second is an array of items to iterate over, and the third is the
function that takes two items and returns a "combined" item.

In this example the array of items is generated using the **map()** function. It takes in the *seriesTotals*
array and returns a new array of objects who's key is the day of the week and the value from *seriesTotals*.
The reduce combine function basically ensures that each time it returns an object it has the day of the week
as a key (and ensures it exists), and incrementally adds each series value to the previous value.
The result is a structure that looks like this.

{% highlight javascript %}
{
    "Monday": 229,
    "Tuesday": 335,
    "Wednesday": 282,
    "Thursday": 258,
    "Friday": 176,
    "Saturday": 119
}
{% endhighlight %}

Now armed with this data the last piece is to determine which day has the highest value.
We saw how we can do that with a couple of tracking variables and a loop. Here is one way
using a **max()** function to do the same thing. The **max()** function takes three arguments.
The first is a starting value, the second is an array or object (object in our case), and the third
is a function that will compare two items and return which item is the "largest".

{% highlight javascript %}
/*
 * Get the day with the most number of items
 */
var topDay = Util.max({ key: "", value: 0 }, totalsByDay, function(a, b) {
    return (((a.value > b.value) ? a : (a.value === b.value) ? a : b));
});
{% endhighlight %}

Whew that was a lot of info. Again I must provide a disclaimer. I am a total n00b at this,
so if you see discussion points here I'd love to hear about it. Mostly I am
enjoying sharing the ride of learning new stuff!

Cheers, and happy coding!