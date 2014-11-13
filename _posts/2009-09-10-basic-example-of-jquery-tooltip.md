---
layout: post
title: Basic Example of jQuery Tooltip
date: 2009-09-10 08:45:00
author: Adam Presley
status: Published
tags: development jquery coldfusion javascript
slug: basic-example-of-jquery-tooltip
---
Oftentimes we developers overlook the simplest solutions to a given
problem. And I must confess that I am the worst, often looking for the
most complex, reusable, scalable solution to something as simple as
*"How do I display a list of promotions in a tabular format while
showing them details of the promo?"*  
  
As I sat down to tackle this simple issue I began the process of mulling
over what types of interfaces I could use to provide the end user a list
of promotions ("Crazy Christmas Promo", for example), all while letting
them quickly see what actually makes up the promotion. For example, our
"Crazy Christmas Promo" may include 20% off t-shirts, free shipping for
orders over $50, and buy 1 Santa hat get one free. The page I am
working on is simply the "manager" page that displays the list of
current promotions and their effective dates. So you would see something
like this:  
  
Promo Name            | Start Date | End Date
----------------------|------------|-----------
Crazy Christmas Promo | 12/01/2009 | 12/30/2009

  
That's cool, but with this view the user can't see the various pieces
that make up the promotion. After a few minutes contemplation it occurs
to me that the best solution is sometimes the simplest. How about I just
add a nifty hovering tooltip when you mouse over the promotion name that
shows what the promo is made up of? Simple, yet effective.  

So with this in mind I did a quick search for jQuery tooltip plugins and
came across [Jrn Zaefferer's jQuery Tooltip plugin](http://bassistance.de/jquery-plugins/jquery-plugin-tooltip/). 
Ok, so the name isn't terribly creative, but the plugin, I soon discovered, is easy to
use and gets the job done.  
  
To begin let's build a quick set of tables (and populate them) that will
house a structure to demonstrate what we are doing. Please note that
this structure is not sound, nor normalized, and it is for demonstration
purposes only.  

{% highlight sql %}
create table promo
(
	promoId int unsigned not null AUTO_INCREMENT primary key,  
	promoName varchar(50) not null,  
	dateStart datetime,  
	dateEnd datetime  
) engine=MyISAM;

create table promoItems
(
	promoItemId int unsigned not null auto_increment primary key,  
	promoId int unsigned not null,  
	appliesTo varchar(50) not null default 'cart',
	discountAmt decimal(10, 2),  
	discountType varchar(25) not null default 'percent',  
	categoryId int unsigned not null default 0
) engine=MyISAM;

insert into promo (promoName, dateStart, dateEnd) values
('Crazy Christmas Promo', '2009-12-01 00:00:00', '2009-12-30 00:00:00');  

insert into promoItems (promoId, appliesTo, discountAmt, discountType, categoryId) values
(1, 'category', 10, 'percent', 1),  
(1, 'category', 12, 'percent', 2),  
(1, 'cart', 10, 'percent', 0);
{% endhighlight %}

With this structure we have a series of promotion items that describe
what the item applies to (cart or category), and how much discount is
applied. This table links back to a promotion that carries a name and
effective date range. Simple enough. A query against it would look
something like this.  

{% highlight sql %}
SELECT
	p.promoId   
	, p.promoName   
	, p.dateStart   
	, p.dateEnd   
	, pi.promoItemId      
	, pi.appliesTo
	, pi.discountAmt   
	, pi.discountType   
	, pi.categoryId   

FROM promoItems AS pi
	LEFT JOIN promo AS p ON p.promoId=pi.promoId   

ORDER BY
	p.promoId,   
	pi.appliesTo
{% endhighlight %}

Cool. Let's take a look at some HTML and CF that will use this query to
display the promotion, and craft a string that tells us what items this
promotion applies to.  
  
{% highlight cfm %}
<cfquery name="qryPromo" datasource="coldfusion">
	SELECT
		p.promoId   
		, p.promoName   
		, p.dateStart   
		, p.dateEnd   
		, pi.promoItemId      
		, pi.appliesTo
		, pi.discountAmt   
		, pi.discountType   
		, pi.categoryId   

	FROM promoItems AS pi
		LEFT JOIN promo AS p ON p.promoId=pi.promoId   

	ORDER BY
		p.promoId,   
		pi.appliesTo
</cfquery>

<html>
<head>
	<script language="javascript" src="jquery.tooltip.min.js"></script>
	<script language="javascript" src="jquery.js"></script>
	<link href="jquery.tooltip.css" type="text/css" rel="stylesheet" />
</head>

<body>
	<table>
		<tr>
			<th>Promo Name</th>
			<th>Start Date</th>
			<th>End Date</th>
		<tr>
		<cfoutput query="qryPromo" group="promoId">
			<cfset title = "Promotion applies to: <ol>" />

			<cfoutput group="appliesTo">
				<cfset title &= "<li>#qryPromo.appliesTo#</li>" />
			</cfoutput>

			<cfset title &= "</ol>" />

			<tr>
				<td title="#title#">#qryPromo.promoName#</td>
				<td>#dateFormat(qryPromo.dateStart, "mm/dd/yyyy")#</td>
				<td>#dateFormat(qryPromo.dateEnd, "mm/dd/yyyy")#</td>
			</tr>
		</cfoutput>
	</table>
</body>
</html>
{% endhighlight %}

The first thing to note is that we are
including the tooltip script, but we are not quite using it yet. The
second thing to note is how we are using ColdFusion's excellent group
feature of a CFOUTPUT loop. If you don't know why, run the query and
you'll see we get three records. We only want to display ONE promo in
the table, and show the two items this promotion applies to (i.e. cart
and category). Notice we are also crafting HTML inside the *title*
variable to apply to the title attribute of a table column.   
  
Now for the fun part. Time to apply some Javascript that puts the
tooltip in place. Basically we want to display a tooltip that tells us
what items this promotion applies to when we hover over the promotion
name. Are you ready? Are you sure?  
  
{% highlight javascript %}
<script language="javascript">
	$(document).ready(function() {
		$('td').tooltip();
	});
</script>
{% endhighlight %}

Wait... don't blink! You might miss it. Yes, that was it. Grab all TD
tags and run the tooltip method. This will apply the tooltip to any TD
with a title attribute. Man I love me some jQuery!  
  
Here is the code sample as a whole. Enjoy, and happy coding!  
  
{% highlight cfm %}
<cfquery name="qryPromo" datasource="coldfusion">
	SELECT
		p.promoId   
		, p.promoName   
		, p.dateStart   
		, p.dateEnd   
		, pi.promoItemId      
		, pi.appliesTo
		, pi.discountAmt   
		, pi.discountType   
		, pi.categoryId   

	FROM promoItems AS pi
		LEFT JOIN promo AS p ON p.promoId=pi.promoId   

	ORDER BY
		p.promoId,   
		pi.appliesTo
</cfquery>

<html>
<head>
	<script language="javascript" src="jquery.tooltip.min.js"></script>
	<script language="javascript" src="jquery.js"></script>
	<link href="jquery.tooltip.css" type="text/css" rel="stylesheet" />
</head>

<body>
	<table>
		<tr>
			<th>Promo Name</th>
			<th>Start Date</th>
			<th>End Date</th>
		<tr>
		<cfoutput query="qryPromo" group="promoId">
			<cfset title = "Promotion applies to: <ol>" />

			<cfoutput group="appliesTo">
				<cfset title &= "<li>#qryPromo.appliesTo#</li>" />
			</cfoutput>

			<cfset title &= "</ol>" />

			<tr>
				<td title="#title#">#qryPromo.promoName#</td>
				<td>#dateFormat(qryPromo.dateStart, "mm/dd/yyyy")#</td>
				<td>#dateFormat(qryPromo.dateEnd, "mm/dd/yyyy")#</td>
			</tr>
		</cfoutput>
	</table>
</body>

<script language="javascript">
	$(document).ready(function() {
		$('td').tooltip();
	});
</script>
</html>
{% endhighlight %}