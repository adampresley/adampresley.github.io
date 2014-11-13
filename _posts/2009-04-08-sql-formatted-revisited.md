---
layout: post
title: SQL Formatted Revisited
date: 2009-04-08 11:30:00
author: Adam Presley
status: Published
tags: development regex
slug: sql-formatted-revisited
---
I posted an entry yesterday about using a regular expression to reformat
SQL queries in code. After a bit more exploring and playing around in
RegexBuddy (wonderful tool) I came up with a better regular expression
to do the job. This version accounts for the last column much better
than the previous expression I tried. So, giving a query like the
following we can apply a search and replace using the following regular
expression: ***(\\s\*)(.+)(?=,|(?\<!,)\\x0D\\x0A|\\z),?***. We then
specify our replace string as***\$1, \$2***. Here's the screen shot.

![SQL Column Formatting using RegexBuddy](http://s3.amazonaws.com/www.adampresley.com/posts/regex-sql-column-replace-1.jpg)
