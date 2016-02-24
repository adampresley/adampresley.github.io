---
author: "Adam Presley"
date: 2010-08-31T13:29:00Z
tags: ["development", "coldfusion", "couchdb"]
title: "My Resume on ColdFusion + FW/1 + CouchDB"
slug: "my-resume-on-coldfusion-fw1-couchdb"
---

The last few days I decided I wanted to play around with Apache's
CouchDB, one of the more popular database engines in the No-SQL
movement. For those who have never heard of it [CouchDB](http://couchdb.apache.org/) is a database
engine written in Erlang, and is an Apache Foundation project. One of
the neatest things about CouchDB is that the API is based entirely on
RESTful JSON services, so all command to it are HTTP commands like PUT,
GET, POST, and DELETE. All data is stored as JSON objects known as
"documents". When you have a chance take a look at it.

For me the project I always fall back to when I want to try something
new is my Resume project. I've kept my resume online for several years
now. My first version of it was to proof-of-concept my PHP framework,
and this worked well for a long time. I then modified it to work with
ColdFusion, Groovy, and Hibernate with MySQL as the persistence layer.
That was fun too. Now I set out to redo this in ColdFusion with CouchDB
as the persistence layer.

The result is a ColdFusion application using Sean Corfield's excellent
[FW/1](http://fw1.riaforge.org/) framework, my [CouchDB4CF](https://sourceforge.net/projects/couchdb4cf/) Java facade, and Apache CouchDB.
Check it out at <http://resume.adampresley.com>. I'll post more soon
about how it is built.

Notice: This site is no longer available