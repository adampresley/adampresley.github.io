---
layout: post
title: Using jqGrid with ColdFusion
date: 2010-03-19 09:17:00
author: Adam Presley
status: Published
tags: development jquery coldfusion javascript
slug: using-jqgrid-with-coldfusion
---

When looking for a grid solution compatible with jQuery that does not
involve using CFGRID, the best I found has been jqGrid. It has a very
impressive list of features, wide support for various incoming data
formats, and is an all around solid solution. For us ColdFusion
developers, however, just trying to "plug it in" and go may leave you
scratching your head. So I am here to hopefully get you started using
this nifty grid in your next ColdFusion application.

The very first order of business is to download jqGrid. For simplicity
sake I would start by downloading the whole set of files. Once you've
done this you must include them on your page. There are four files you
will concern yourself with to start.

{% highlight cfm %}
<script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/1.4.2/jquery.js"></script>
<script type="text/javascript" src="/js/jqgrid/i18n/grid.locale-en.js"></script>
<script type="text/javascript" src="/js/jqgrid/jquery.jqGrid.min.js"></script>

<link rel="stylesheet" href="/js/jqgrid/ui.jqgrid.css" type="text/css" />
{% endhighlight %}

The above code shows that we need to at least include the jQuery
library, the English locale settings for jqGrid, the jqGrid JavaScript
source, and the appropriate CSS file. The next order of business is to
create a container for jqGrid to live in. For the grid we need to create
a simple, blank table like so.

{% highlight html %}
<table id="myGrid"></table>
{% endhighlight %}

That takes care of the markup and necessary includes, so now let's jump
to the ColdFusion side. Server side we need to take some set of data and
return it in a JSON format that jqGrid likes. First let's make a simple
CFM page that gives us the JSON output we need for jqGrid.

{% highlight cfm %}
<cfsetting showdebugoutput="false" />

<cfset data = queryNew("id,name,title,age") />

<cfset queryAddRow(data) />
	<cfset querySetCell(data, "id", 1) />
	<cfset querySetCell(data, "name", "Adam Presley") />
	<cfset querySetCell(data, "title", "Giant nerd") />
	<cfset querySetCell(data, "age", 31) />

<cfset queryAddRow(data) />
	<cfset querySetCell(data, "id", 2) />
	<cfset querySetCell(data, "name", "Steve Good") />
	<cfset querySetCell(data, "title", "Flex dude") />
	<cfset querySetCell(data, "age", 30) />

<cfset queryAddRow(data) />
	<cfset querySetCell(data, "id", 3) />
	<cfset querySetCell(data, "name", "Adrian Moreno") />
	<cfset querySetCell(data, "title", "Kung foo") />
	<cfset querySetCell(data, "age", 21) />

<cfset queryAddRow(data) />
	<cfset querySetCell(data, "id", 4) />
	<cfset querySetCell(data, "name", "Chris Jordan") />
	<cfset querySetCell(data, "title", "Mac daddy") />
	<cfset querySetCell(data, "age", 32) />

<cfset queryAddRow(data) />
	<cfset querySetCell(data, "id", 5) />
	<cfset querySetCell(data, "name", "Will Belden") />
	<cfset querySetCell(data, "title", "Look how awesome I am") />
	<cfset querySetCell(data, "age", 33) />

<cfset queryAddRow(data) />
	<cfset querySetCell(data, "id", 6) />
	<cfset querySetCell(data, "name", "Todd Pires") />
	<cfset querySetCell(data, "title", "Coldbox Voodoo Magic!") />
	<cfset querySetCell(data, "age", 20) />

<cfset queryAddRow(data) />
	<cfset querySetCell(data, "id", 7) />
	<cfset querySetCell(data, "name", "Tim Jesperson") />
	<cfset querySetCell(data, "title", "Mr. Limo") />
	<cfset querySetCell(data, "age", 22) />

<cfset result = {} />

<cfset result["rows"] = [] />
<cfset result["page"] = javaCast("int", 0) />
<cfset result["records"] = javaCast("int", data.recordCount) />
<cfset result["total"] = javaCast("int", 1) />

<cfloop query="data">
	<cfset temp = {} />
	<cfset temp["id"] = javaCast("int", data.id) />
	<cfset temp["cell"] = [] />

	<cfset arrayAppend(temp["cell"], javaCast("int", data.id)) />
	<cfset arrayAppend(temp["cell"], data.name) />
	<cfset arrayAppend(temp["cell"], data.title) />
	<cfset arrayAppend(temp["cell"], javaCast("int", data.age)) />

	<cfset arrayAppend(result["rows"], temp) />
</cfloop>

<cfoutput>#serializeJson(result)#</cfoutput>
{% endhighlight %}

The first part of this code (gridjson.cfm) is simple. We are just
crafting a query by hand for demonstration purposes. The result
structure could use a bit of explanation. jqGrid expects a few things in
the JSON response. First it expects that all the grid data be in an
array named **rows**. Each item in this array is an object that contains
two keys: **id** and **cell**. The **id** key can be thought of as the
primary key value for the record. The **cell** key contains an array of
all the record's values. In our example we include **id**, **name**,
**title**, and **age**. Also notice we include keys for **page**,
**records**, and **total**. These keys are used for grid paging, which
we will not investigate in this posting. I'd also like to point out that
jqGrid by default is expecting a certain format of JSON to come back.
This is why we are so explicit in how we are crafting the JSON response.
Also note I am using *array notation* when building my structures. Why?
Because the ColdFusion method **serializeJson** will maintain key case
when array notation is used. If you do not use array notation, the
structure keys are capitalized. JavaScript is a case-sensitive language
don't forget, so maintaining case is pretty important. Now that we have
a valid JSON response coming back, let's build some JavaScript to
actually build the grid. The grid works by using a jQuery selector
against the table we created, and calling the **jqGrid** method. This
method potentially takes a *lot* of parameters, but we will only concern
ourselves with the few necessary to get a grid working. Take a look.

{% highlight javascript %}
$("#myGrid").jqGrid({
	url: "gridjson.cfm",
	datatype: "json",
	height: 165,
	width: 450,
	mtype: "POST",
	emptyrecords: "No employees to display.",
	colNames: [ "Name", "Title", "Age" ],
	colModel: [
		{ name: "name", width: 45, sortable: false, editable: false },
		{ name: "title", width: 65, sortable: false, editable: false },
		{ name: "age", width: 35, sortable: false, editable: false }
	]
});
{% endhighlight %}

As you can see we select the table *myGrid* and call the **jqGrid**
method against it. From here we specify our *gridjson.cfm* page we
crafted above to supply the JSON data for the grid. We also make sure to
specify that the grid is expecting JSON as the response by setting the
**datatype** to "json". The other important pieces are the **colNames**
and **colModel** keys. **colNames** are what header columns are actually
*displayed* in the grid. **colModel** is the fun part. It is an array of
objects, each object representing a column in the JSON data. Here the
**name** key is the name of the column coming back. The **width** key
simply specifies how wide this column should display. **sortable** and
**editable** are to determine if the column can be sorted or edited. Put
it all together, and you have a working jqGrid! Have fun with jQuery,
and happy coding!
