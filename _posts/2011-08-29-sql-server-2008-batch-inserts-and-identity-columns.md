---
layout: post
title: SQL Server 2008 Batch Inserts and Identity Columns
date: 2011-08-29 22:39:00
author: Adam Presley
status: Published
tags: development sql
slug: sql-server-2008-batch-inserts-and-identity-columns
---
As most are aware SQL Server 2008 supports batch inserts, much like
MySQL has for a billion years. Basically this is the ability to insert
multiple records in a single statement like so.

{% highlight sql %}
INSERT INTO people (
	firstName
	, lastName
	, age
) VALUES
	( 'Adam', 'Presley', 32),
	('Joe', 'Schmo', 38);
{% endhighlight %}

That feature of course rocks, but I wasn't quite sure how to then
retrieve all my new IDs on an identity column for this batch of inserts.
Turns out this isn't too hard. See the highlighted lines in the example
below, and you will note that this is pretty simple actually. Happy
coding!

{% highlight sql %}
DECLARE @results TABLE(
	id INT NOT NULL
);

INSERT INTO people (
	firstName
	, lastName
	, age
)
OUTPUT INSERTED.id INTO @results
VALUES
	('Adam', 'Presley', 32),
	('Joe', 'Schmo', 38);

SELECT id FROM @results;
{% endhighlight %}
