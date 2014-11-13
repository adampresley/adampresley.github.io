---
layout: post
title: Regular Expressions - Commas at the Beginning of Column Names
date: 2009-04-06 08:47:00
author: Adam Presley
status: Published
tags: development regex
slug: regular-expressions-commas-at-the-beginning-of-column-names
---
If you've worked with a lot of inline SQL like I have over the years
you've probably discovered quickly that as you have to add more columns
to your query, **especially** columns that are only added on a dynamic
condition, it can be easy to mangle up your commas. Here's an example of
a common type of query I deal with at work on a regular basis.  

{% highlight sql %}
SELECT
    column1,
    column2,
    column3
FROM table1
{% endhighlight %}

Now if I am asked to add a column based on some condition it could look
like this.  
  
{% highlight cfm %}
SELECT
    column1,
    column2,
    column3
    <cfif someCondition EQ true>, column4</cfif>
FROM table1
{% endhighlight %}

So over time I've come around to the SQL syntax where you put commas
**before** your column name, making this kind of problem less of an
issue.  

So in working with old queries I've had to convert a few to this syntax
just for readability. Here is a little regular expression I cooked up
that can help this along. It's not quite perfect, and doesn't work in
Eclipse (lack of lookahead support perhaps?), but will work in more
powerful regex editors (Boxer, for example). It doesn't catch the last
column, and will double up on commas that already at the beginning of
the column. Like I said, not perfect, but helpful nonetheless.  
  
{% highlight text %}
(\s*)([a-z0-9\._\(\)\s',]+)(?=,),
{% endhighlight %}

Use this regex as your search condition, and the replace text is "**\$1,
\$2**", without quotes.  
  
Happy coding!
