---
layout: post
title: Escape of the single quote
date: 2006-08-18 09:54:00
author: Adam Presley
status: Published
tags: development
slug: escape-of-the-single-quote
---

Discovered the most interesting and strange bug in ColdFusion MX 7. Not
sure about 6, cause I tested in 7. Anywho. With the MX release
ColdFusion automagically escapes single quotes when using variables in
the CFQUERY tag.  
  
It came to my attention at work that we were getting errors on a page
that were related to single quotes not being escaped. I found this to be
odd, so I took a look at the code. Interestingly enough were spitting a
string to the an update SQL statement that was a function call (i.e.
'#someFunction(variable)#').  
  
After some experimenting I discovered that ColdFusion fails to escape
single quotes in this VERY condition. ONLY in update statements. Not
select. Not insert. Only update. Odd.
