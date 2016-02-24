---
author: "Adam Presley"
date: 2010-09-10T16:39:00Z
tags: ["development",  "javascript"]
title: "Function to Add/Stack JavaScript Events"
slug: "function-to-addstack-javascript-events"
---

I'm in the process of modifying pages in a ColdFusion application where
I do not have the option of plugging in jQuery. ExtJs is available in
certain parts of the application, but getting it in others is not always
an option. As such I needed to attach additional click events to buttons
on a form that may already have an existing click event. As such I
needed a quick way to add a new one without overwriting the old click
event.

To do this I first created a namespace to put my new method into so I
don't trample on any existing implementation. I will call this namespace
"mine". :) Then I crafted a function to add events to an element. I
wanted this method to be pretty versatile, and it needs the element to
attach the event to, the name of the event, a handler, and an optional
scope argument. The handler just needs to be a method to call when the
event fires. The scope argument is so you can override what scope the
handler is called against. This is useful if you need the handler to
execute in the scope of another object or function. I also wanted the
ability to pass in the element as either a string name, or the actual
element object. Let's look at the code.

{{< highlight javascript >}}
var mine = mine || {};

/**
 * Add an event handler. Will check to see if there are
 * already existing event handlers registered, and if so
 * will execute them.
 * @author Adam Presley
 * @param el DOM element or string of DOM element name
 * @param handler A function to execute when the click event occurs
 * @param scope The scope in which to execute this handler. The default
 *    is to execute in the scope of the DOM element
 */
mine.addEvent = function(el, eventName, handler, scope) {
   var o = (typeof el == "object") ? el : document.getElementById(el);
   var currentEvent = (o["on" + eventName]) || undefined;

   /*
    * Assign the onclick event.
    */
   o["on" + eventName] = function(e) {
      if (currentEvent !== undefined) {
         if (scope === undefined) currentEvent.call(o, e);
         else currentEvent.call(scope, e);
      }

      if (scope === undefined) handler.call(o, e);
      else handler.call(scope, e);
   }
}
{{< / highlight >}}

The very first thing of note is the declaration of **mine**. If it
is already defined use that, otherwise define a blank object. This is
our namespace that the **addEvent** method will be defined in.

Once inside the **addEvent** method the first order of business is
to get the DOM element. If you pass a string name into **el** it
will grab the element by ID for you. Otherwise if you pass in the DOM
element it just uses that. From here we then check to see if there is
already a registered event on this object based on the **eventName**
you pass in.

After this we assign a method to handle the requested event. If there
was already an event this new handler calls the old event so we can keep
existing event handlers. It then will call your newly passed in
**handler**. By default the passed in **handler** is called in
the scope of the DOM element, but you may override it if you wish. Here
is an example of calling this.

{{< highlight javascript >}}
mine.addEvent("btnSubmit", "click", function(e) {
   alert("You clicked me!");
});
{{< / highlight >}}

In almost all cases I would recommend using a library that does all this
for you because they've gone through the trouble of cross browser
testing and so on. However, when necessary it sure is handy to know how
to do this the old fashioned way. Happy coding!
