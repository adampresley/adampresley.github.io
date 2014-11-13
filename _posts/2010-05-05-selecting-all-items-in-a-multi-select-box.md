---
layout: post
title: Selecting All Items in a Multi-Select Box
date: 2010-05-05 11:56:00
author: Adam Presley
status: Published
tags: development jquery javascript
slug: selecting-all-items-in-a-multi-select-box
---
Here's a quick [jQuery](http://jquery.com) tidbit. If you ever
need to select or unselect all items in a muti-select box
(a \<select\> tag set to allow multiple selected items), here's
a jQuery selector to do it.

{% highlight javascript %}
$("#selectBox *").attr("selected", "selected");
{% endhighlight %}

Notice the asterisk after the ID selector? That tells jQuery to get
*ALL* child elements of the select box. We then set the **selected**
attribute to **selected**, and suddenly all the items are highlighted!
To unselect them all you would do the opposite.

{% highlight javascript %}
$("#selectBox *").attr("selected", "");
{% endhighlight %}

Setting the attribute **selected** to blank unselects them all. Man I
love jQuery!!

Happy coding!
