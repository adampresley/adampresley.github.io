---
author: "Adam Presley"
date: 2009-02-10T22:00:00Z
tags: ["general", "development", "csharp"]
title: "Consuming Web Services in .NET 3.5 - Web References"
slug: "consuming-web-services-in-net-35-web-references"
---

I have to admit I feel a little bit behind the curve. I've not goofed
too much with the .NET 3.5 specific stuff much lately, and much of my
experience has been in .NET 2.0. But today I packed up the laptop and
headed to Starbucks to get out of the office so I could have some peace
and quite, as the office is usually the wrong place for that kind of
thing.

For the last couple of hours I've been working on supplemental
documentation for the Clearview DataServices API. This has been a pet
project of mine for our company for the last year and a half, and in the
last month the CTO has really decided to make a major marketing push and
get this polished up and to our clients. I've already got 70+ pages of
the API's function documentation written, but I have found that the
average developer contracted to use my API for our clients don't
understand the business well, so when they consult the documentation to
try to do anything with our system, they end up lost.

With this in mind I'm writing two things. The first is a "Cookbook"
style document which gives common problems or tasks, and the code to
solve the problem. I've read these kinds of books myself in the past and
I'm personally a big fan of them. The second thing I am writing is a set
of libraries to simplify consuming our web service API from ColdFusion,
PHP, and .NET.

Apparently in .NET they've made some changes to how web service
references are added. Once upon a time you simply added a "Web
Reference" to the project, providing the URL and a namespace, and the
proxy classes were generated for you. I go to do this now, and there's
new stuff there. I no longer see the ability to add a "Web Reference",
but now a "Service Reference".

For those of you who don't know, when you enter the address to your web
service, there is a new "Advanced" button at the bottom of the dialog.
Click on this, and you will see a new button on a *second* dialog to add
a "Web Reference'. Viola! They've hidden it!! Chances are this is what
you will be looking for.
