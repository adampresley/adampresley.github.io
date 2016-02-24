---
author: "Adam Presley"
date: 2008-12-09T10:34:00Z
tags: ["development", "sql"]
title: "How to Change the Owner of a SQL 2005 Table"
slug: "how-to-change-the-owner-of-a-sql-2005-table"
---

It isn't often that I need such a thing, but occasionally I get asked by
an engineer to change the owner of a database table in MS SQL 2005. This
happens sometimes, though I cannot always remember how, but changing the
owner of a table is pretty easy. Just execute the following script,
making sure you are logged in as "sa".

{{< highlight sql >}}
exec sp_changeObjectOwner 'originalObjectOwnerAndName', 'newOwner';
{{< / highlight >}}

For example, if the object name is "MyTable1", and is owned by
"DOMAIN1\User1", and you wish it to simply be "dbo", then you would do
the following:

{{< highlight sql >}}
exec sp_changeObjectOwner 'DOMAIN1\User1.MyTable1', 'dbo';
{{< / highlight >}}

Pretty simple.
