---
layout: post
title: INSERT or UPDATE in MySQL
date: 2011-08-25 23:25:00
author: Adam Presley
status: Published
tags: coldfusion sql mysql
slug: insert-or-update-in-mysql
---
Tonight I **finally** got to use the very awesome *ON DUPLICATE KEY
UPDATE* feature in MySQL. If you are wondering this is a great feature in
MySQL that allows you, in one statement, to insert a record, and if that
record already exists, update it instead. The criteria for already
existing is based on checking primary and unique keys on the record.  
  
The only question that wasn't clearly answered by MySQL's documentation
for me was how *autoincrement*fields behave. Turns out this is pretty
easy. For demonstration purposes assume that we have a table with *id*,
*firstName*, *lastName*, and *age*. The ColdFusion snippet below will
not only insert a new record if one does not already exist, it will
update an existing one if an existing ID is passed in. The sample
function takes a single structure as an argument.  
  
    ::cfm
    <cffunction name="savePerson" output="false">
        <cfargument name="person" />
        
        <cfset var qrySave = "" />

        <cfquery name="qrySave" datasource="#application.dsn#">
            INSERT INTO persons (
                id
                , firstName
                , lastName
                , age
            ) VALUES (
                <cfqueryparam value="#arguments.person.id#" cfsqltype="CF_SQL_INTEGER" />
                , <cfqueryparam value="#arguments.person.firstName#" cfsqltype="CF_SQL_VARCHAR" maxlength="50" />
                , <cfqueryparam value="#arguments.person.lastName#" cfsqltype="CF_SQL_VARCHAR" maxlength="50" />
                , <cfqueryparam value="#arguments.person.age#" cfsqltype="CF_SQL_INTEGER" />
            ) 
            ON DUPLICATE KEY UPDATE
                id=LAST_INSERT_ID(id)
                , firstName=<cfqueryparam value="#arguments.person.firstName#" cfsqltype="CF_SQL_VARCHAR" maxlength="50" />
                , lastName=<cfqueryparam value="#arguments.person.lastName#" cfsqltype="CF_SQL_VARCHAR" maxlength="50" />
                , age=<cfqueryparam value="#arguments.person.age#" cfsqltype="CF_SQL_INTEGER" />;

            SELECT LAST_INSERT_ID() as newId;
        </cfquery>

        <cfset arguments.person.id = qrySave.newId />
        <cfreturn arguments.person />
    </cffunction>

The highlighted lines are of interest. Line 18 is where the magic
happens. This is what tells MySQL that we want to update instead of
insert when an existing record is found. Line 19 is a trick that will
set the LAST_INSERT_ID() variable in MySQL to the existing ID in the
event of an update. Line 24 simply gets the last inserted ID, and if
this turned out to be an update, gets the ID from line 19.  
  
Please note that if the *id*column is an autoincrement, and you pass a
non-existent ID, that's OK. It will simply ignore what you pass in to
it, and generate a new ID.  
  
MySQL == nice. Happy coding!
