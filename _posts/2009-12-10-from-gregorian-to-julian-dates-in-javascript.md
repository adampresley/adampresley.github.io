---
layout: post
title: From Gregorian to Julian Dates in JavaScript
date: 2009-12-10 04:30:00
author: Adam Presley
status: Published
tags: development javascript
slug: from-gregorian-to-julian-dates-in-javascript
---

I am working on a contract right now where I am having to do some
manipulation of dates in a calendar. This particular calendar stores
dates in Julian Day Number (JDN) format. This by itself is not a serious
issue as PHP has simple functions to handle these conversions. However I
have a situation where I have a Gregorian date in the DOM, and I need to
pass the Julian date to a JavaScript function.   
  
After a bit of research, and a few algorithms online that didn't do the
trick, I used a formula from a site called [Astronomy Answers](http://www.astro.uu.nl/~strous/AA/en/reken/juliaansedag.html) that
seems to have done the trick.  
  
Below is a piece of code I have written to convert a Gregorian date to a
Julian Day Number, as well as a sample of using it.  

    :::javascript
    Date.prototype.ToJulian = function() {
    var d = this.getDate();
    var y = this.getFullYear();
    var m = this.getMonth() + 1;

    return Math.floor((1461 * (y + 4800 + (m - 14) / 12)) / 4 + (367 * (m - 2 - 12 * ((m - 14) / 12))) / 12 - (3 * ((y + 4900 + (m - 14) / 12) / 100)) / 4 + d - 32075);
    };

    var testDate = "12/9/2009";
    var parsedDate = new Date(Date.parse(testDate));
    var julianDate = parsedDate.ToJulian();

    alert(julianDate);

Let me know if you run into any issues with this calculation, or find a
better way to do it. Also let me know if you can get the reverse to work
(going from Julian to Gregorian), as this is giving me some issues!
