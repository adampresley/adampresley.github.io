---
author: "Adam Presley"
date: 2009-12-17T16:41:00Z
tags: ["development", "coldfusion", "openbd"]
title: "SES URLs with OpenBD and Google App Engine"
slug: "ses-urls-with-openbd-and-google-app-engine"
---

A little tidbit that I though I'd share. While playing with the Google
App Engine (GAE) version of the OpenBD CFML engine I noticed that SES
friendly URL's weren't working for me. After a small search I came
across a bug entry on the OpenBD site that suggested that the SES filter
is not setup in the GAE version by default, and that the following XML
needed to be added to the WEB-INF/web.xml configuration file.

{{< highlight xml >}}
<filter>
	<filter-name>SearchEngineFriendlyURLFilter</filter-name>
	<display-name>SearchEngineFriendlyURLFilter</display-name>
	<description>SearchEngineFriendlyURLFilter</description>
	<filter-class>com.newatlanta.filters.SearchEngineFriendlyURLFilter</filter-class>
	<init-param>
		<param-name>extensions</param-name>
		<param-value>cfm,cfml</param-value>
	</init-param>
</filter>
<filter-mapping>
	<filter-name>SearchEngineFriendlyURLFilter</filter-name>
	<url-pattern>/index.cfm/*</url-pattern>
</filter-mapping>
{{< / highlight >}}

This is to be inserted just before the closing tag. After a quick
restart of the OpenBD context, I was good to go.

Happy coding!
