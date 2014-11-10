---
layout: post
title: Triggering the Change Event on a Select Box Programatically In jQuery
date: 2009-09-23 12:26:00
author: Adam Presley
status: Published
tags: development jquery javascript
slug: triggering-the-change-event-on-a-select-box-programatically-in-jquery
---

Don't know if anyone else has had to do this, and it seems so simple,
but it took me a couple of minutes to find this. I have a ColdFusion
page with a lot of jQuery magic happening with a popup dialog that
allows the user to edit address information. This dialog has a list of
countries that, once one is selected, will perform an AJAX call to get a
list of states for the country, and adjust the state select box
accordingly.

In a situation where I wanted the user to click Edit, I perform an AJAX
call to get the address information. I then have to populate all of the
input boxes with the values from the JSON response.

	:::javascript
	$('#country').val(jsonResponse.country);

As you may or may not know if you simply set the value of the countries
select box, it doesn't fire off a change event, so the above code is not
effective. After a little bit of searching and reading the jQuery
documents, the following snippet of code will fire the event, and any
subsequent listeners you have bound to that event will execute.

	:::javascript
	$("#country").val(jsonResponse.country);
	$("#country").trigger("change");

That **trigger()** method fires off the actual event, and all is groovy.
Happy coding!
