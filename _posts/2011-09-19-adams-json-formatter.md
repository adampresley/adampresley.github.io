---
layout: post
title: Adam's JSON Formatter
date: 2011-09-19 13:05:00
author: Adam Presley
status: Published
tags: development jquery json programming coldfusion
slug: adams-json-formatter
---
[Curious Concept's JSON Formatter](http://jsonformatter.curiousconcept.com/) 
is a great JSON formatter and validator application. I found myself working 
with a JSON dataset that was larger than what this tool allowed, so naturally 
I set out and made my own. [Adam's JSON Formatter](http://jsonformatter.adampresley.com/) 
runs on [CloudBees](http://www.cloudbees.com/), a PaaS platform for running Java applications. 
This little tool is built on OpenBD and uses the excellent [JSONlib](http://json-lib.sourceforge.net/) 
tool for JSON beautification. For the front end stuff I'm using jQuery and jQuery UI. 
CloudBees was super easy to get started with, and super easy to deploy my application to. 
And of course the tiny footprint of OpenBD is perfect for the small, 128M slice
you get for free with CloudBees. So I recommend you give them a try.
That and OpenBD. :) Cheers, and happy coding!
