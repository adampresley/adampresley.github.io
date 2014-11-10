---
layout: post
title: Issues Serializing ColdFusion Session Object
date: 2009-06-24 18:26:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: issues-serializing-coldfusion-session-object
---

At work I've been asked to brainstorm solutions to a problem where a
user's session and application information is lost when they get kicked
off of one server and are moved to another. Essentialy we have a load
balancer with "sticky session" support, so a user, once logged into our
application, goes onto a web server, and stays there. Should that web
server crash, however, they will get moved to the next web server. That
next server, however, has no knowledge of this user's application and
session information, thus forcing them to log in to our application
again. This isn't optimal.  
  
Now, before you ask, I have already been told that, in these difficult
economic times, we will **NOT** be purchasing ColdFusion Enterprise to
handle sessions across a cluster. This is why I am working on a
"home-grown" solution, despite the fact that Adobe has already gone
through the trouble. I digress.  
  
So here's the concept. When a request is finished, serialize the SESSION
scope (and a few others, but we'll conveniently ignore that for now) to
some type of persistent, or semi-persistent medium, such as a database,
or remote cache server like [Apache Jakarta JCS](http://jakarta.apache.org/jcs/index.html) (which looks cool
BTW). That's fine and all, but the problem is that the SESSION object,
which is effectively a Java HashTable, must be transformed, or
serialized, into some format that can be stored.  
  
WDDX is a fairly attractive option, as it is built in, works, and can
turn the SESSION object into XML. There are two factors that deter me
from this, however. The first is that the XML produced by CFWDDX is huge
and bloated, and our SESSION scope isn't exactly small. The second is
performance. I'm concerned that serializing and deserializing to and
from this format could be time consuming for EVERY request.  
  
It was at this point that I decided to delve into the Java documentation
and see what was available. I came across the java.io.ObjectOutputStream
class. This class is specifically designed to transform objects into a
binary format suitable for transport and storage. Sweet. So I cooked up
a sample that creates a SESSION object, puts some values in it,
structure and arrays, and then serializes it to a byte array stream so I
can dump it to a page and see what it looks like. From there I wanted to
deserialize it BACK to the SESSION scope. Let's take a look at this code
and the resulting page.  
  
    :::cfm
    <cfset session.key1="value1">
    <cfset session.key2="value2">
    <cfset session.key3="value3">
    <cfset session.somethingcomplex="arrayNew(1)">
     <cfset session.somethingcomplex[1]="array1">
     <cfset session.somethingcomplex[2]="array2">
     <cfset session.somethingcomplex[3]="array3">
     
    <cfset session.another="structNew()">
     <cfset session.another.hey1="hey1">
     <cfset session.another.hey2="hey2">

    <cfoutput>
     

    <cfset "java.io.bytearrayoutputstream").init()="" outbytestream="createObject(&quot;java&quot;,">
    <cfset "java.io.objectoutputstream").init(outbytestream)="" oos="createObject(&quot;java&quot;,">

    <cfset oos.writeobject(session)="">
    <cfset oos.close()="">


    <cfset serialized="outByteStream.toByteArray()">

    <strong>SESSION as a byte array (#outByteStream.size()# bytes).</strong>
    <cfdump expand="false" var="#serialized#">




    <cfset "java.io.bytearrayinputstream").init(serialized)="" inbytestream="createObject(&quot;java&quot;,">
    <cfset "java.io.objectinputstream").init(inbytestream)="" ois="createObject(&quot;java&quot;,">

    <cfset deserialze="ois.readObject()">
    <cfset deserialze)="" structappend(session,="">


    <strong>Restored SESSION scope: </strong>
    <cfdump expand="false" var="#SESSION#">

As you can see in the first dump we have a binary representation of the
SESSION object. And the second dump is quite promising with one
exception. The array in the SESSION object did not deserialize properly.
Interestingly I started digging around and discovered other people that
have run into the very same issue, with no apparent resolution.  
  
The good news here is that I think we might be able to get away with not
storing arrays in our SESSION, but I'm hoping CF 9 addresses this issue.
