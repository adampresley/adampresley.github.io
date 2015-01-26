---
layout: post
title: JavaScript Multiselect Widget, With No jQuery!
date: 2015-01-21 12:33
author: Adam Presley
tags: development javascript
comments: true
---
Last night I decided to write a multi-select JavaScript widget. I need such a widget to allow the users of a little app I am working on to select one or more items. I wanted it to look a little more pretty than the default *&lt;select&gt;* box and I also didn't want to force users to have to hold CTRL to select more than one item. I did a little searching around the internet and found a number of controls that do exactly what I want, but all of them required an external library, such as jQuery, Angular, and even Mootools. This particular project is **not** using jQuery on purpose which makes those libraries out of the question. So I start writing my own.

![Screenshot](/assets/adampresley/images/posts/multiselect-1.png)

<!-- excerpt -->

The widget (currently) attaches an object to the *window* object called **multiselect**. To use it you simply call the **render()** function with the ID of an element to render to, and data to drive the elements that get rendered for selection. Inside **render()** an outer *<div>* element is created, followed by a *<ul>*. I then iterate over the data collection passed in with the config to render *&lt;li&gt;* elements. Finally **render()** returns an object to the caller so they can do things like return an array of selected items. Calling it looks something like the code below. In this example I am rendering the widget into a *&lt;div&gt;* named **food** and am providing a set food types as data. Then I am attaching a *click* event to a button to log the selected items to the console.

{% highlight javascript %}
var foodSelector = window.multiselect.render({
    elementId: "food",
    data: [
        { "value": 1, "text": "Greek" },
        { "value": 2, "text": "Thai" },
        { "value": 3, "text": "American", "selected": true },
        { "value": 4, "text": "Mexican" }
    ]
});

document.getElementById("getSelected").addEventListener("click", function() {
    console.log(foodSelector.getSelected());
    console.log(foodSelector.getSelectedIndexes());
    console.log(foodSelector.getSelectedValues());
});
{% endhighlight %}

What was fun about this was really digging into what modern JavaScript and CSS can do without the aide of common libraries like jQuery or AngularJS. I clearly have missed much in the last couple of years! You can do nearly anything jQuery can do with just plain, vanilla JavaScript. For example, doing query selectors?

{% highlight javascript %}
var allWithThisClass = document.querySelectorAll(".thisClass");
{% endhighlight %}

Or how about array iteration using the *for each* style?

{% highlight javascript %}
var a = [1, 2, 3];
a.forEach(function(i) {
	console.log(i);
});
{% endhighlight %}

Great right? JavaScript has come a *long* way! If you are interested in checking out my widget, or even just the code you can find it on Github. If JavaScript package management is more your speed it can also be found on Bower.

* Github - [https://github.com/adampresley/multiselect](https://github.com/adampresley/multiselect)
* Bower Search - simple-multiselect

Here are some links that helped me out and you might find useful.

* [You Might Not Need jQuery](http://youmightnotneedjquery.com/)
* [DOM Selection Without jQuery](http://youmightnotneedjquery.com/)
* [Mozilla Developer Network](https://developer.mozilla.org/en-US/)

Cheers, and happy coding!
