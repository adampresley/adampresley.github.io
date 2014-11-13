---
layout: post
title: Basic Query Loop and Group in ColdFusion
date: 2009-12-30 11:14:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: basic-query-loop-and-group-in-coldfusion
---
While going over the [DreamInCode.net](http://www.dreamincode.net) forums I 
came across this question.   

> Good day everyone. I'm hoping someone can point me in the right
> direction. I've been using Coldfusion for awhile, although I admit,
> I'm not a master at it.  
>   
> I'm pulling my hair out on a project I'm hoping someone can shed some
> light on.  
>   
> I have a table in my database that I want to group and output a report
> on.  
>   
> It's a table tracking purchases and types of purchases.  
>   
> Here's the data I'm collecting: from_name, from_company,
> transaction_type, and amount. I have a few other fields, but they
> aren't needed for this report. There are multiple transactions per
> person and a varying number of persons.  
>   
> What I need to do is to create queries that will group by name or
> company, which doesn't matter, give me a count of transaction types (3
> of A, 6 of B, etc.) as well as a total amount of each type, then show
> the total transactions count by name or company and the total amount.  
>   
> I'm trying to output in a table having each row be the data from each
> company.  
>   
> Can someone point me in the right direction to group such output, do
> counts on the number of records as well as show totals dollars?  
>   
> Thank you so much for any advice you might have.

  
ColdFusion has one of the best grouped looping language constructs in
any language I've used to date. The ability to query some data, loop
over the data grouping by a column, then further looping over all items
that fall under that group, is a very powerful feature.  
  
I setup a small MySql database that houses data like the question above
describes. Here is some SQL to help you get setup and started.  
  
{% highlight sql %}
create table groupingexample (
	id int not null primary key auto_increment,
	from_name varchar(255),
	from_company varchar(255),
	transaction_type varchar(50),
	amount decimal(10,2)
);

insert into groupingexample
	( from_name, from_company, transaction_type, amount )
values
	( 'Adam Presley', 'AdamPresley.com', 'Type A', 100.00 ),
	( 'Bob Barker', 'The Price is Right', 'Type B', 124.20 ),
	( 'Adam Presley', 'AdamPresley.com', 'Type A', 23.15 ),
	( 'Joe Keyy', 'Company 1', 'Type B', 100.00 ),
	( 'Yasmine Ippy', 'Yasmine\'s House', 'Type B', 100.00 ),
	( 'Yasmine Ippy', 'Yasmine\'s House', 'Type B', 10.70 ),
	( 'Adam Presley', 'AdamPresley.com', 'Type B', 25.00 );
{% endhighlight %}

Now that we have that, we need a query. When looping over data that
needs to be grouped it is important to remember to order your data
results by the columns you are grouping by. In our case we are going to
group by **from_company**, so we will order by that column first.  

{% highlight cfm %}
<cfquery name="qryTransactions" datasource="cfexamples">
	SELECT
		  ge.id
		, ge.from_name
		, ge.from_company
		, ge.transaction_type
		, ge.amount

	FROM groupingexample AS ge
	ORDER BY
		ge.from_company,
		ge.from_name,
		ge.transaction_type
</cfquery>
{% endhighlight %}

To loop over a data set and group by a specific column we will use the
tag, and with it the **group** attribute. In the **group** attribute we
will put in the column name of the column we wish to group by. In our
case we will use **from_company**. Inside of this loop if you just
dropped an H1 tag with the company name you will see only four records
displayed. This is because of the grouping.  

{% highlight cfm %}
<cfoutput query="qryTransactions" group="from_company">
	<h1>#qryTransactions.from_company#</h1>
</cfoutput>  
{% endhighlight %}

To now loop over the items under each company we do another tag, this
time with no attributes. Inside of this loop we will start tracking
sub-totals, totals, and displaying who paid what. Let's look at that
now.  
  
{% highlight cfm %}
<cfquery name="qryTransactions" datasource="cfexamples">
	SELECT
		  ge.id
		, ge.from_name
		, ge.from_company
		, ge.transaction_type
		, ge.amount

	FROM groupingexample AS ge
	ORDER BY
		ge.from_company,
		ge.from_name,
		ge.transaction_type
</cfquery>

<cfset tracking = structNew() />
<cfset tracking.totalAmount = 0.00 />

<cfoutput query="qryTransactions" group="from_company">
	<h1>#qryTransactions.from_company#</h1>

	<cfset tracking.transactionTypes = structNew() />
	<cfset tracking.subTotalAmount = 0.00 />

	<ul>
		<cfoutput>
			<li>#qryTransactions.from_name# paid #dollarFormat(qryTransactions.amount)# (transaction type #qryTransactions.transaction_type#)</li>

			<cfset tracking.subTotalAmount = tracking.subTotalAmount + qryTransactions.amount />
			<cfset tracking.totalAmount = tracking.totalAmount + qryTransactions.amount />

			<!---
				If we have tracked this transaction type already
				increment it. If not, add the type to the structure
				and set it to one.
			--->
			<cfif structKeyExists(tracking.transactionTypes, "#qryTransactions.transaction_type#")>
				<cfset tracking.transactionTypes["#qryTransactions.transaction_type#"] = tracking.transactionTypes["#qryTransactions.transaction_type#"] + 1 />
			<cfelse>
				<cfset tracking.transactionTypes["#qryTransactions.transaction_type#"] = 1 />
			</cfif>
		</cfoutput>
	</ul>


	<!---
		Display sub-totals.
	--->
	<strong>Subtotal Amount: #dollarFormat(tracking.subTotalAmount)#</strong>
	<strong>Transaction Type Counts:</strong>
	<ul>
		<cfloop collection="#tracking.transactionTypes#" item="type">
			<li>#type#: #tracking.transactionTypes["#type#"]#</li>
		</cfloop>
	</ul>
</cfoutput>

<strong>Grand Total:</strong> <em><cfoutput>#dollarFormat(tracking.totalAmount)#</cfoutput></em>

<cfdump var="#qryTransactions#" />
{% endhighlight %}

And there you have it. Grouped looping is a super-cool powerful feature
of ColdFusion. Happy coding!  
