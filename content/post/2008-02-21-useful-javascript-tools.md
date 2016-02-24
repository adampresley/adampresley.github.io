---
author: "Adam Presley"
date: 2008-02-21T08:02:00Z
tags: ["development", "javascript"]
title: "Useful Javascript Tools"
slug: "useful-javascript-tools"
---

I've been doing a lot of development using ExtJS, which involves a lot
of AJAX work. And for anyone who has ever used AJAX you know you must
deal with JSON. JSON is fine and dandy, and is certainly the best way to
get compact data from a remote call response, but it sure can be hard to
read at times. Anyone using this stuff will also tell you that when
doing a lot of Javascript work that finding errors can be challenging.
This is why I recommend the following suite of tools.

* [JSON Formatter](http://www.curiousconcept.com/jsonformatter/)
* [JSLint](http://www.jslint.com/)
* [Firebug](http://www.getfirebug.com/) for Firefox
* [Debug Bar](http://www.debugbar.com/?langage=en) for IE

JSON formatter is a nifty tool that allows you to either paste JSON code
into a textbox, or provide a URL, and it will spit out nice, neatly
formatted JSON. That's right. All spacing, line breaks, etc... all
formatted for easy readability.

JSLint is a great tool for figuring out those hard syntax related bugs
when attempting to write clean Javascript code. The same premise applies
here. Paste in your code, and it will spit out a list of exceptions and
problems, complete with line numbers and descriptions on how to resolve
the issues.

Firebug is simply the best Javascript debugging tool available today.
This tool allows you to view header content, responses, error messages
and line numbers, included scripts and CSS, DOM selection using your
mouse cursor, and even Javascript step-by-step execution!

Debug Bar is like Firebug, except that it is for IE, and it features
only a small subset of what Firebug does. If you are in an environment
where IE is required, however, it proves useful to see what has
executed, what headers where sent, and responses.
