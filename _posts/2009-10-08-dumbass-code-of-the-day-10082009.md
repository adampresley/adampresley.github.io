---
layout: post
title: Dumbass Code of the Day - 10/08/2009
date: 2009-10-08 12:55:00
author: Adam Presley
status: Published
tags: development coldfusion dumbass
slug: dumbass-code-of-the-day-10082009
---
Welcome to another edition of "Dumbass Code". Here's a snippet that's
fun. This bit of code gets a list of product IDs, iterates over them,
and if the product ID is greater than zero, and less than 999,999 then
it is a "special" type of product and is not counted. In this case a
"special" product is a shipping record, or handling, etc... And "not
counted" is done by using the **listRest** ColdFusion method to remove an
item. I hadn't even HEARD of this method till now, and it turns out that
it just removes the first element from a list. Okaaaayyy...  

{% highlight cfm %}
<!-- this gets a count of actual products -->
<cfset prodlist = ValueList(getProds.product_id) >
<cfif listlen(prodlist) gt 0>
	<cfloop index="thisprodid" list="#prodlist#">
		<cfif 999999 lt evaluate(thisprodid) or evaluate(thisprodid) lt 0 >
			<cfset prodlist = listrest(prodlist)>
		</cfif>
	</cfloop>
</cfif>

<cfset numprods = listlen(prodlist)>
{% endhighlight %}

Kind of odd eh? Seems like a lot of unnecessary processing to just get a
count. How about something like this?  

{% highlight cfm %}
<!--- Get a count of actual products --->
<cfset numProducts = 0 />

<cfloop query="getProds">
	<cfif getProds.product_id GT 0 && getProds.product_id LT 999999>
		<cfset numProducts++ />
	</cfif>
</cfloop>
{% endhighlight %}
  
Hmmm.
