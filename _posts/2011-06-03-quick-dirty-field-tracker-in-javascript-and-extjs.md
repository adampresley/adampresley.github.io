---
layout: post
title: Quick Dirty Field Tracker in JavaScript and ExtJS
date: 2011-06-03 13:39:00
author: Adam Presley
status: Published
tags: development javascript
slug: quick-dirty-field-tracker-in-javascript-and-extjs
---

Here's a quick tidbit. I was asked if I had a quick way to track changes
on an old, existing page in an application. Once a Cancel button was
clicked, or the user tried to close the browser, the developer wanted to
determine if anything changed, and alert the user if there was a
change.  
  
So I cooked up this quick snippet to track those changes using
ExtJS/Sencha and JavaScript.  
  
{% highlight javascript %}
window.dirty = {
    items: [],
    add: function(itemToCheck) {
        var found = false;

        Ext.each(this.items, function(item, index) {
            if (item.id === itemToCheck.id) found = true;
        });

        if (!found) this.items.push(itemToCheck);
    }
};

Ext.each(Ext.query("input, select, textarea"), function(item, index) { 
    Ext.get(this).on("change", function() { 
        window.dirty.add(this);
    });
});
{% endhighlight %}
  
This bit of code will add hooks for an onChange event for all select,
input and textarea boxes on a page and track if they get changed from
their original value. Then all the developer has to do is check for
*window.dirty.items.length > 0* to see if there are "dirty" fields on
the page.  
  
Happy coding!
