---
layout: post
title: Setting Cookies then Relocating in ColdFusion
date: 2009-12-15 06:56:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: setting-cookies-then-relocating-in-coldfusion
---

This question came across the ColdFusion User Group just recently. The
developer has some type of login form, followed by a query that
validates the user's information. It then attempts to set a cookie then
redirect to another page. The subsequent page checks for the existence
of the cookie, and if it is not set will relocate back to the login
page. The developer kept getting kicked back to the login page, as if
the cookie was never set.

In this case it wasn't. The reason is because when you do a CFLOCATION
existing headers will be cleared out, and the Location header will be
sent instead, cause the browser to redirect the user to a new page. The
cookie header will be completely ignored!

Here is a sample on how to set the cookie, make sure the client browser
gets the cookie, **THEN** redirect.

	:::coldfusion
	<cfset getPageContext().getOut().clearBuffer() />
	<cfcookie name="testingCookies" value="#form.userName#" />
	<cfflush />

	<cflocation url="postLogin.cfm" />

Happy coding!
