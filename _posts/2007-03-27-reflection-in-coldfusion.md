---
layout: post
title: Reflection in ColdFusion
date: 2007-03-27 08:35:00
author: Adam Presley
status: Published
tags: development coldfusion reflection
slug: reflection-in-coldfusion
---
So I was reading a blog entry on Ben Nadel's site about him trying to do
function pointers in ColdFusion. It seems that a solution was never
presented to him, though I am not sure as the entry is nearly a month
old. I decided to play with the concept and met with some success. Using
reflection in Java I was able to dynamically bind to and call a
ColdFusion function. I only had success with the functions that had only
string parameters, and had failure with functions that had integer
arguments. I suspect it is a problem with the class type. Not sure. But
here is an example of using reflection in ColdFusion to call the COMPARE
function.  

**NOTE**: Code missing
  
Interesting, eh?

