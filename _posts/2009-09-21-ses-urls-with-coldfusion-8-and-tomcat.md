---
layout: post
title: SES URLs with ColdFusion 8 and Tomcat
date: 2009-09-21 05:54:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: ses-urls-with-coldfusion-8-and-tomcat
---

A little tidbit that I had to work on today. To make sure that I am
properly testing my applications in both Railo AND ColdFusion, I bundled
up ColdFusion 8 as a WAR file and deployed it to Tomcat. That went
easily enough. When I browsed to my Coldbox application, however, I was
greeted with an error telling me it couldn't find the location.

Having gone through this with Railo I immediately start searching for
some solutions via Google. I did find one that indicated that the
web.xml file in my WEB-INF directory in my application context would
contain servlet settings already in the file, but they are commented
out. Sure enough, somewhere around line 403, it was there.

However, they are not supported by Tomcat, and I did go through the
trouble of trying. To make my Coldbox application work I ended up adding
the following mapping:

	:::xml
	<servlet-mapping id="coldfusion_mapping_6">
		<servlet-name>CfmServlet</servlet-name>
		<url-pattern>/index.cfm/*</url-pattern>
	</servlet-mapping>

Restart the context, and we are in great shape! Happy coding!
