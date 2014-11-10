---
layout: post
title: Parsing the URL using JavaScript
date: 2010-04-04 16:18:00
author: Adam Presley
status: Published
tags: development javascript
slug: parsing-the-url-using-javascript
---

Here's a quick little post regarding a question I got today. I was asked
how one could parse the URL to get the parameters and do something with
them.   

The first order of business is to know that the URL address can be found
in a variable tied to the *window* object called *location*. The
property that has the actual URL is named *href*. The next thing we
should know is that URL parameters always follow the page name on the
URL, and come after a question mark. Each parameter is a key/value pair,
where the key is the name of the parameter, followed by an equal sign,
then by the value for that parameter. So if we wished to pass my name
over the URL as a set of first and last name parameters, you would see
something like the following:
http://www.adampresley.com/index.php?firstName=Adam&lastName=Presley.
That URL will have a parameter named *firstName* with the value of
**Adam**, and a parameter named *lastName* with the value of
**Presley**. Pretty easy right?   

To parse the URL and get those parameters we will make use of a function
built into JavaScript called **split**. The **split** method
works against a string of characters, and takes an argument that tells
the code what character(s) to split on. In our case we want to split the
URL at the question mark. Why? Cause we care about everything
**AFTER** the question mark. From there we can further use the
**split** function to divide the parameters by the ampersand, and
then by the equal sign. Let's take a look at what that code looks like.
  
    :::javascript
    var url = window.location.href;
    var urlParts = url.split("?");

    /*
     * If we have anything after the question mark, parse it
     */
    if (urlParts.length > 1)  {
        var rightSide = urlParts[1];
        var outerParts = rightSide.split("&");

        for (var outer = 0; outer < outerParts.length; outer++) {
            var innerParts = outerParts[outer].split("=");
            var key = innerParts[0];
            var value = (innerParts.length > 1) ? innerParts[1] : "";

            /*
             * Do something with the key/value information 
             * here. Add what you need.
             */
            alert("Key = " + key + " value = " + value);
        }
    }

As you can see in the code we aren't actually doing anything with the
retrieved key and value, but you could use it to do whatever you needed
in JavaScript. In the case of the individual that asked me for this
code, they needed to redirect dynamically to another HTML page based on
the URL parameter.   

So there you have a simple way to parse URL parameters. Happy coding!!

