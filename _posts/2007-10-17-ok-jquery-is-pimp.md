---
layout: post
title: Ok, jQuery Is Pimp
date: 2007-10-17 20:07:00
author: Adam Presley
status: Published
tags: development jquery javascript
slug: ok-jquery-is-pimp
---

Alright, I'm with [Ben Nadel](http://www.bennadel.com) on this one... jQuery is **PIMP!!**
I started playing with it, and although I have barely scratched the
surface I can tell that this thing is really powerful. As an example let
us say that I have a form with a bunch of text boxes that I want to
clear. Let's say that the id of the form is *frmSearch*. Here's how that
could be done in jQuery.  
  
{% highlight javascript %}
$("#frmSearch :text").val("");
{% endhighlight %}

How cool is that? Yes, one line. Here's what that does. The dollar sign
(*$*) is the shortcut for the jQuery class. Inside the parenthesis we
say to match an element who's ID is *frmSearch*. Then the *:text* tells
jQuery that we want to match **ALL** text boxes in that form. Then we
are calling the *val* function which gets or sets the value of an
element. In this case we are setting the value to blank.  
  
Yeah, you need to check it out.
