---
author: "Adam Presley"
date: 2010-05-05T21:17:00Z
tags: ["development", "sql"]
title: "How To Insert Multiple Records in one Insert Statement in MSSQL"
slug: "how-to-insert-multiple-records-in-one-insert-statement-in-mssql"
---

A friend showed me this trick a year or two ago, and it has come in
handy once again. In Microsoft SQL Server 2005 and higher if you need to
insert more than one record into a database table in a single statement
(kind of like how MySQL has), here is how you can do it.

{{< highlight sql >}}
INSERT INTO contacts (
    firstName
    , lastName
    , title
)
SELECT
    'Adam'
    , 'Presley'
    , 'Senior Software Engineer'

UNION ALL

SELECT
    'Ben'
    , 'Nadel'
    , 'ColdFusion Awesome Dude';
{{< / highlight >}}

Notice that this solution starts off with a basic INSERT statement, then
uses a SELECT statement to provide the values. The trick here is to do a
**UNION ALL** for each additional record you wish to insert. Pretty cool
eh? Happy coding!!

