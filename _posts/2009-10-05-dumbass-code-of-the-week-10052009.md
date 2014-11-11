---
layout: post
title: Dumbass Code of the Week - 10/05/2009
date: 2009-10-05 12:06:00
author: Adam Presley
status: Published
tags: development coldfusion dumbass
slug: dumbass-code-of-the-week-10052009
---

Today's **Dumbass Code of the Week** is brought to you by code from my
current job. This code, written circa 2002, is real, and has not been
modified or harmed in any way during the posting of this blog entry. Are
you ready for it??  

{% highlight cfm %}
<cfif Find("@", attributes.sEmail) GT 0>
	<cfif Find(".", attributes.sEmail, Find("@", attributes.sEmail)) GT Find("@", attributes.sEmail)>
		<!--- Set the Message the user will see if the process does not succed. --->
		<cfset UserMessage = "<p>A password could not be found for the email address #attributes.sEmail#.</p>">
	<cfelse>
		<cfset attributes.sEmail = "Invalid Email Address">
		<cfset UserMessage = "<p>Invalid Email Address.</p>">
	</cfif>
<cfelse>
	<cfset attributes.sEmail = "Invalid Email Address">
	<cfset UserMessage = "<p>Invalid Email Address.</p>">
</cfif>
{% endhighlight %}

Take a good long look at it. What will the outcome of this "Lost
Password" functionality be? Mmmmm, tasty.
