---
layout: post
title: A Prettier Query to JSON for ColdFusion using Groovy
date: 2009-09-22 07:46:00
author: Adam Presley
status: Published
tags: development groovy coldfusion sql
slug: a-prettier-query-to-json-for-coldfusion-using-groovy
---

ColdFusion 8 introduced the ability to serialize data structures,
arrays, and queries to JSON notation, making building AJAX-enabled
applications even easier. There are a couple of annoyances, however,
that still persist, and simply drive me nuts. Take the following query
as an example.  
  
{% highlight cfm %}
SELECTs.state_id, s.stateName, s.stateAbbrevFROM states AS sORDER BY s.state
{% endhighlight %}

When you use the [serializeJson()](http://livedocs.adobe.com/coldfusion/8/htmldocs/functions_s_03.html) method in ColdFusion, you get a
JSON structure that looks something like this.  
  
{% highlight javascript %}
{"COLUMNS":["STATE_ID","STATENAME","STATEABBREV"],"DATA":[[52,"Alabama","AL"],[53,"Alaska","AK"],[54,"Arizona","AZ"]]}
{% endhighlight %}

No, that isn't the entire US state list because I truncated it for
demonstration purposes. What you see here, however, is a key named
"DATA" that is an array or arrays. Each sub-array is the actual row of
data. Also notice that everything is capitalized. Ick. Now I don't know
if you are like me, but I like mine to look a little something more like
this.  
  
{% highlight javascript %}
[{"stateName":"Alabama","stateAbbrev":"AL","state_id":52.0},{"stateName":"Alaska","stateAbbrev":"AK","state_id":53.0},{"stateName":"Arizona","stateAbbrev":"AZ","state_id":54.0}]
{% endhighlight %}

In this example we have an array of objects. Each object is a row of
data, and the keys are the column names. Also note that the column names
are cased the way I like them, and not capitalized like ColdFusion tends
to do. Here's how.  
  
By using a little [Groovy](http://groovy.codehaus.org/) and my modification to allow [binding the
arguments collection](|filename|/cfgroovy2-and-arguments-scope-in-functions) we can use the metadata from the query's
underlying [ResultSet](http://java.sun.com/j2se/1.5.0/docs/api/java/sql/ResultSet.html) Java object to get the column names as they
were sent to the SQL server, preserving case. Check it out.  
  
{% highlight cfm %}
def result = []
def metadata = arguments.query.getMetaData()
def numCols = metadata.getColumnCount()

while (arguments.query.next()) {
	row = [:]
	(1..numCols).each {
		index ->

		def colName = metadata.getColumnName(index)
		def value = arguments.query.getString(index).toString()
		row.put(colName, (arguments.query.wasNull()) ? "" : value)
	}

	result.add((coldfusion.runtime.Struct) row)
}

arguments.result = result;
{% endhighlight %}

To break this down we start with a blank array called *result*. Then we
need to get the query's metadata using the **getMetaData()** method, as
well as how many columns we have by calling **getColumnCount()**.  
  
Then we iterate over query rows by calling the **next()** method. In the
loop we create a blank structure that will represent a row of data. We
then iterate from 1 to the number of query columns, grab the column name
and value, and put it into the *row* structure by name. After we have
our row built we add it to the *result* array. The final piece is to
call ColdFusion's own **serializeJson()** method to turn this array of
structures into JSON.  
  
Viola! Happy coding!
