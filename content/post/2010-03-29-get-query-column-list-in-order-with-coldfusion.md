---
author: "Adam Presley"
date: 2010-03-29T04:47:00Z
tags: ["development", "coldfusion"]
title: "Get Query Column List In Order with ColdFusion"
slug: "get-query-column-list-in-order-with-coldfusion"
---

Don't know if you've ever noticed, but it has been a long time behavior
that ColdFusion will alphabetize the order of columns in a query result
set. If you are working with queries in a manner where order is
important, such as serializing them to a format in JSON not supported
natively by ColdFusion, this becomes quite a bother. Below is a simple
solution to handle working with the query and its columns in order they
were put into the SQL statement.

{{< highlight cfm >}}
<cfset metadata = qry.getMetaData() />
<cfset rowIndex = 0 />
<cfset index = 0 />
<cfset cols = [] />

<!---
    Get an array of column names in ORDER they
    were put into the SQL statement.
--->
<cfloop from="1" to="#metadata.getColumnCount()#" index="index">
    <cfset cols[index] = metadata.getColumnName(index) />
</cfloop>

<!---
    Now we can loop over the query in column order
--->
<cfset rows = [] />
<cfloop from="1" to="#qry.recordCount#" index="rowIndex">
    <cfset column = {} />
    <cfloop from="1" to="#arrayLen(cols)#" index="index">
        <cfset column["#cols[index]#"] = qry["#cols[index]#"][rowIndex] />
    </cfloop>

    <!---
        Now do something with this new row of data.
        In this example we are simply making an
        array of structs. You could do something fancier.
    --->
    <cfset arrayAppend(rows, column) />
</cfloop>
{{< / highlight >}}

In the above example we aren't transforming the data into anything more
fancy than an array of structures, but you could obviously turn the data
into any type of format you wished that could then be passed through
*serializeJson* to make JSON, or perhaps turned to XML. In any case,
if column order is important to you, this should help out.
Happy coding!
