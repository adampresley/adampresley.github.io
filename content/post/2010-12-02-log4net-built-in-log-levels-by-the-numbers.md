---
author: "Adam Presley"
date: 2010-12-02T22:05:00Z
tags: ["development", "csharp"]
title: "Log4Net - Built in Log Levels by the Numbers"
slug: "log4net-built-in-log-levels-by-the-numbers"
---

I had the pleasure of using the .NET port of the very popular and
powerful log4j library called [log4net](http://logging.apache.org/log4net/index.html).
I've found it as powerful and enjoyable as log4j and it has **greatly**
simplified my logging needs for this C# project.

At some point I found myself needing to create a custom log level. This
is actually documented fairly well, and there are even a couple of
examples provided in the download. What I couldn't find easily, however,
is the numeric value assigned to the built-in log levels. Ya know, WARN,
INFO, DEBUG, etc... Why is this important? Because when you build a
custom log level you specify both a name and an integer value for the
level. This is so that the level names can be sorted, and when you
configure an appender you can specify what level to log at. That
comparison is done by the integer value assigned to the log level name.
But for the life of me I couldn't find the integer values for the
built-in levels!

Well, dig in the source code I did, and down in the **/Core/Level.cs**
file I found what I needed. Here is a quick table of the built-in
levels.

* Off - int.MaxValue
* Emergency - 120,000
* Fatal - 110,000
* Alert - 100,000
* Critical - 90,000
* Severe - 80,000
* Error - 70,000
* Warn - 60,000
* Notice - 50,000
* Info - 40,000
* Debug - 30,000
* Fine - 30,000
* Trace - 20,000
* Finer - 20,000
* Verbose - 10,000
* Finest - 10,000
* All - int.MinValue

Now armed with this knowledge I now know where I can place my custom log
levels. Happy coding!
