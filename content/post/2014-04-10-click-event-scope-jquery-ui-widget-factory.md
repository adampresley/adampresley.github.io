---
author: "Adam Presley"
date: 2014-04-10T02:42:00Z
tags: ["development", "jquery", "programming", "javascript"]
title: "Click Event Scope in jQuery UI Widget Factory"
slug: "click-event-scope-jquery-ui-widget-factory"
---

A little tidbit that I figured out today regarding the scope of **this** in a jQuery UI dialog button. Actually I feel foolish for not checking the actual value of **this** sooner, but you can't win them all. With the creation of the **WidgetFactory** I've been looking at writing my UI widgets using this tool. In fact the search widget on this blog uses the jQuery UI **WidgetFactory**. Originally I thought I had an issue with scope, but turns out I was mistaken. You see the search widget extends the jQuery UI **dialog** object and has a button on it. Usually when an event handler is called the scope of **this** references the DOM element that fired the event. So I assumed that **this** would reference the button being clicked. Fortunately I was wrong.

When jQuery UI handles a click event on buttons defined in your options object it actually changes the scope of **this** to the dialog element itself. This is a good thing. Why? Because you may be initializing your widget against multiple matched DOM elements in your jQuery selector, so you need to know which dialog/widget you are working with. So for anyone who may have wondered, there you have it.