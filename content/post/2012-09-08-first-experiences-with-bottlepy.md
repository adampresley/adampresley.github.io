---
author: "Adam Presley"
date: 2012-09-08T10:55:00Z
tags: ["programming", "python", "bottle"]
title: "First Experiences With Bottlepy"
slug: "first-experiences-with-bottlepy"
---

Here lately I've taken to writing a bit more code in Python. It isn't my
first time around in this lovely language, but for the last few months
I've spent more time in Groovy and Grails, and I felt it was time to
revisit the language where whitespace matters. Most of my experience
with Python up until this point revolves around scripts and utility
code, such as install automation. I had made up my mind to try working
on a web project this time, and naturally I started with the one web
framework that I had heard most about, and had dabbled with before,
Django.

Being a full stack framework similar to Grails, where
I have spent a lot of time recently, I figured I would be comfortable
here. Truth be told though I am not. Perhaps it is a phase, or maybe I'm
just not cut out for full stack web frameworks, but it seems that every
time I work in one I end up leaving for a less featured framework.
Coldbox, Zend, CakePHP, Grails... all of them are big and have
everything you'd ever need, and if they don't they support plugins for
everything else. This is great, but in the end I always seem happier
moving to the smaller frameworks. FW/1, Gaelyk (for GAE), and now Bottle
for Python.

Bottle is a micro-framework for Python web development. We are talking a
single, 128K file that supports URL routes, view templates, JSON
serialization, built-in web server for development, and plugins. It's
super light weight and fast, and it doesn't take a lot of heavy reading
to get started. As an example here is a simple view definition.

{{< highlight python >}}
from bottle import view, route, request
from app.model.core.adampresley.Factory import Factory

@route("/home")
@view("home")
def home():
   viewData = {}
   service = Factory().getVisitorService()

   viewData["numVisits"] = service.getVisitorCount()
   return viewData
{{< / highlight >}}

The template engine is actually just a light wrapper around Python code
itself with basic include ability. Compared to other template engines
I've used it is pretty utilitarian yet it seems to serve my purposes so
far. For example:

{{< highlight html >}}
<h1>Welcome!</h1>
% if numVisits > 0:
   <p>You are visitor number {{numVisits}}!</p>
% else:
   <p>No visitors before you! You are the first!</p>
%end
{{< / highlight >}}

I can honestly say I am enjoying Bottle, and look for more posts on
this nifty framework soon. Happy coding!
