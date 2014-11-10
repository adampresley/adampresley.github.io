---
layout: post
title: Dumbass Code of the Week 12-20-2009
date: 2009-12-21 11:11:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: dumbass-code-of-the-week-12-20-2009
---

Please, someone explain to me why this was necessary. The first line of
code is just a sample setup. The real code has a loop over some orders,
assembles this string, and adds the word "NewOrder" at the end of it. It
then executes line #2 with the revers, replace, and reverse. This is
the part I'm not sure I get. I run it, and see that the word NewOrder is
just removed. So why all the crazy reverses and such?? No idea.  
  
	:::cfm
	<cfset OrderDetailStringM = "1,5,SplitHere,10,600,NewOrder5,85,684" />
	<cfset OrderDetailStringM = reverse(replace(reverse(OrderDetailStringM),"redrOweN",""))>

	<cfdump var="#OrderDetailStringM#" />

Uh...
