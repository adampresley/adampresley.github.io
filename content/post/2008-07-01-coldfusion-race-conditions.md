---
author: "Adam Presley"
date: 2008-07-01T19:39:00Z
tags: ["development", "coldfusion"]
title: "ColdFusion Race Conditions?"
slug: "coldfusion-race-conditions"
---

One of the things I've been struggling with at work lately is related to
bizzare ColdFusion errors. Essentially what I am seeing is a page that
normally works just suddenly throw an error stating that a specific
variable cannot be accessed as an object because it is of type
**java.lang.String**. The problem with this is that it is in fact a CFC
instance, and works great for the most part. Then suddenly it will stop
working, and logging off the software, and back in, is the only
solution. This, of course, is problematic.

A breakdown of the problem so far would possibly
indicate an issue with a thread race-condition. Please note that this is
only a theory, and I haven't had ample time to actually test it. But
consider the following:

* A structure is being stored in the Application scope that basically
  contains permission information for the current user.
* When needed, a component is instantiated that takes this structure
  as an argument, and stores a local variable reference to this
  structure. It will read data from this to determine a user's
  permission against specific application areas/objects.
* Data is also written to this copy of the structure (property of the
  CFC).
* Unless I'm mistaken, structures passed to a function are passed by
  reference (see the [interesting experiment by Ben Nadel](http://www.bennadel.com/blog/516-ColdFusion-Arguments-Object-Can-Act-As-Ghetto-Pass-By-Reference-Array.htm) on his
  blog).

So with this in mind consider that I am passing the structure from the
Application scope to a CFC property (the THIS scope is this particular
case). That means that the CFC, when modifying the structure, is
actually modifying the Application scope version due to the fact that it
was passed in by reference, and is a pointer.

This line of thinking let me to suspect that there was possible memory
trashing due to race-conditions of several places in the application
accessing (both read and write) this variable in the Application scope.
The only thing that seems to trip me up is that if you take a look at
ColdFusion structures, they are based on [java.util.Hashtable](http://java.sun.com/j2se/1.4.2/docs/api/java/util/Hashtable.html), which
is setup for synchronized access. This memory trashing should not
occur... So I have to admit I'm a bit stumped, cause even the CF docs
state that you should put a lock around writes (and even reads on
variables that can be written to simultainously) to avoid memory
trashing.

Overall I still have a lot of testing to do, and any information anyone
has on this would be useful.
