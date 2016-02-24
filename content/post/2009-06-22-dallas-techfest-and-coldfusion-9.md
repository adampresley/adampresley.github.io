---
author: "Adam Presley"
date: 2009-06-22T08:09:00Z
tags: ["technology", "general", "development", "coldfusion"]
title: "Dallas Techfest and ColdFusion 9"
slug: "dallas-techfest-and-coldfusion-9"
---

This last Friday I attended the ColdFusion track at the Dallas Techfest.
I went with a coworker and ex-coworker specifically to see the
ColdFusion 9 preview. It was a good presentation that I must admit I'm a
little excited about.

For me, this is the most exciting ColdFusion release in a long time. In
my eyes ColdFusion, with the version 9 release, is well on its way to
becoming a programmer's language. Allow me to elaborate.

I have a programming background in C/C++. In fact you
could call C++ my first love of sorts. When I got into web-based
development my first exposure was in ASP classic, and although I
consider it clunky, it was programming to me. PHP was my next web
scripting langauge, and is to this day my favorite web-based language to
work with. I develop with ColdFusion for a living, however, and have
always maintained a love/hate relationship to it with all of the
tag-based commands and such. Yet this tag-based concept is a strong
selling point for most CF developers... something I never really
understood.

ColdFusion 9 sets out to bridge the gap between the scripters and the
taggers. Finally. Let's start with the fact that CF 9 will finally have
full language support in CFSCRIPT. About time! Yes, you can now create
components, throw exceptions, and do queries in CFSCRIPT without writing
hand-spun custom function wrappers.

The next exciting feature for me is that components are getting more OOP
goodness. Actual constructors for initializing components, as well as a
supposed "strict" mode on CFCs to force CF to compile them as POJOs
(Plain Old Java Objects). I can only see this as a performance benefit,
as the current cost of calling a method on a CFC involves instantiating
a compiled Java class... for EACH method!

Other notable features includes a true ternary expression, new language
support for the Hibernate ORM library, and more. Looks like a good
release!

Cheers, and happy coding!
