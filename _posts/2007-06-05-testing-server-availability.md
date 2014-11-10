---
layout: post
title: Testing server availability
date: 2007-06-05 06:48:00
author: Adam Presley
status: Published
tags: development
slug: testing-server-availability
---

It's a simple thing, but quite useful. In the organization where I work
we have over ten web servers, so oftentimes the location of a problem
can be difficult to pinpoint. In this line of thought I was asked to
create a ColdFusion page which tries to communicate with our SMTP
server, and spit out **200 OK** if it is available. So here's one
approach.

	:::coldfusion
	<cfsetting showdebugoutput="false">

	<cfscript>
		success = 0;

		try {
			socket = createObject("java", "java.net.Socket").init(
				javacast("string", "mailserver.address.com"),
				25
			);
			socket.close();

			success = 1;
		} catch(Any ex) {
			success = 0;
		}
	</cfscript>

	<cfif success>
		<cfset getPageContext().getOut().clearBuffer() />200 OK<cfabort />
	<cfelse>
		<cfset getPageContext().getOut().clearBuffer() />404 Not Found<cfabort />
	</cfif>

This code simple attempts to establish a socket connection to the mail
server on port 25, and if successful, spits out **200 OK**, otherwise it
reports **404 Not Found**. On failure the ***java.net.Socket*** class
throws an exception.
