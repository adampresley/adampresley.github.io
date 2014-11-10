---
layout: post
title: Groovy 1.8 Released
date: 2011-04-27 10:29:00
author: Adam Presley
status: Published
tags: development groovy
slug: groovy-18-released
---

Groovy 1.8 has finally been released, and let me comment on that with
**YAY!** There are some really great enhancements to an already great
language. I'm only going to point out a few of the areas that I am
particularly interested in, but you can see the release notes [here](http://docs.codehaus.org/display/GROOVY/Groovy+1.8+release+notes).

Of particular interest to me:    

* Command chains for nicer Domain-Specific Languages  
	* This one is cool because it takes the already powerful DSL abilities of Groovy and allows an even more natural language syntax. For example: *paint wall with red, green and yellow*, or *convert result to xml*
* GPars bundled within the Groovy distribution  
	* I'm already working in a project that uses the excellent GPars library, and having it bundled is simply a nice to have.
* Native JSON support  
	* This is great, as I find myself using JSON more frequently than XML these days. Now I don't have to add additional 3rd party libraries.
* New AST Transformations  
	* @Log annotation - YES! Now with just a simple annotation I can have my log4j logger ready and available. Sweet.
	* @ThreadInterrupt - This is particularly handy. Now instead of having to put the logic checks in thread methods and loops for an interruption, this will do it for you!
	* @ConditionalInterrupt - Similar to above, except this will actually interrupt the thread for you when a specified condition is met. This has some possiblity.

Overall those are the items I'm most excited about. If you haven't had a
chance to try Groovy, do it [now!](http://groovy.codehaus.org/)
