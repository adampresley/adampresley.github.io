---
layout: post
title: MSSQL Script to Generate Random SSN
date: 2010-09-26 22:33:00
author: Adam Presley
status: Published
tags: development sql
slug: mssql-script-to-generate-random-ssn
---

If you've ever worked with a database that contains sensitive data you
may have had the task of "scrubbing", or cleaning the data so it can be
used by the average developer and QA engineer. Last week I was working
on setting up a testing server and received a "scrubbed" database from
our database team. They had taken and cleaned up all the sensitive data.
For the social security numbers they had changed each one to
"111-11-1111". This is normally fine, but we ran into a ColdFusion page
that was querying for all duplicate SSNs, and displaying links on the
page to alert a user to go and correct the duplicate entry. This would
normally been innocent enough except that there were over 600,000
entries in the system, and caused Internet Explorer to live up to its
nickname, Internet Exploder.   
  
To correct this I crafted a simple SQL script to randomize the SSNs,
making the data feel a bit more natural, and the page not crash. Here it
is.  

    :::sql
    DECLARE @id INT
    DECLARE @newSSN VARCHAR(11)
    
    DECLARE pcursor CURSOR FOR SELECT
        id
    FROM personTable WHERE 1=1

    OPEN pcursor
    FETCH NEXT FROM pcursor INTO @id

    WHILE @@FETCH_STATUS=0
    BEGIN
        SET @newSSN = (CAST(CAST(100 + (898 * RAND()) AS INT) AS VARCHAR(3)) + '-' + CAST(CAST(10 + (88 * RAND()) AS INT) AS VARCHAR(2)) + '-' + CAST(CAST(1000 + (8998 * RAND()) AS INT) AS VARCHAR(4)))
        PRINT @newSSN

        UPDATE personTable SET
            SSN=@newSSN
        WHERE
            id=@id

        FETCH NEXT FROM pcursor INTO @id
    END

    CLOSE pcursor
    DEALLOCATE pcursor
  
This little script simple gets a cursor for all the people in our table
and iterates over them. For each iteration we create a random SSN value
using the RAND() function and casting them to VARCHARs. We update the
value in the table, grab the next record from the cursor, and
rinse/lather/repeat until we are done.  
  
Happy coding!
