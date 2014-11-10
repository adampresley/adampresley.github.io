---
layout: post
title: Yahoo! Widget Converter Issue
date: 2007-10-16 08:02:00
author: Adam Presley
status: Published
tags: development
slug: yahoo-widget-converter-issue
---

I ran across an interesting problem today. At work I wrote a [Yahoo!
Widget](http://widgets.yahoo.com/) that monitors our web servers' response time and memory
consumption. I had made a few updates and enhancements to it and was
ready to release it to the engineers here that use it.

To do this you use the Yahoo! Widget Covnerter, which is a widget,
interestingly enough. So I dropped the folder onto the widget and
suddenly... BLAM!... error: **"move(): path 'C:\\Windows\\system32' is
not allowed."**. Ok... that's a new one, cause I've used this before.

The fix I found in the converter widget's support thread. Here's how to
fix this strange issue.

* Close Yahoo Widgets completely (i.e. Right Click the system tray icon and select Exit Yahoo Widgets)
* Right-click the Startup folder located in the Start menu under All Programs and select Open.
* Delete the shortcut to YahooWidgetEngine.
* Browse to Program FIles\\Yahoo!\\Widgets and make a new shortcut to YahooWidgetEngine.exe
* Copy this new Shortcut to the Startup folder.
* Start Yahoo Widgets up again and all should be well!
