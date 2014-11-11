---
layout: post
title: SES URLs With Mura on Tomcat
date: 2010-07-02 12:08:00
author: Adam Presley
status: Published
tags: development coldfusion mura
slug: ses-urls-with-mura-on-tomcat
---

Tonight as I'm setting up a new server I wanted to make sure I had URL
rewrite ability. Normally this is no issue as I use Apache for most
anything web server related. However this time I am setting up a Mura
site on Railo running on Tomcat. And yes, I'm using Tomcat for both my
Java Servlet engine and my web server. As such I wanted to ensure I have
"pretty URLs" for my Mura-based application. In this post I will attempt
to show you how to set that up.

At this point I am going to assume you have a running Tomcat and Mura
application. The first thing we need to do is download Paul Tuckey's
excellent UrlRewriteFilter Java filter. This can be located at
<http://www.tuckey.org/urlrewrite/>. The version I downloaded as of this
writing is 3.2.0. Once downloaded you can extract the contents anywhere
you wish to keep it. You will notice there isn't much there... in fact
at the top level there is only a folder named **WEB-INF**. Opening that
folder doesn't reveal much more.

If you have Tomcat running you'll need to stop it temporarily for this
next part. After stopping Tomcat, copy the file
**/WEB-INF/lib/urlrewrite-3.2.0.jar** into your lib folder, found at
**{TOMCAT}/lib**, where **{TOMCAT}** is the folder where you have placed
your Tomcat installation. By copying this JAR file there you are making
the URL re-writer filter available to all Java applications running in
your Tomcat instance.

The next phase is to setup Railo (yes, you can do this in Adobe CF as
well) to use this Java re-writer filter. Where you do this varies based
on how you have setup your Railo application, but I have my Railo files
extracted in my Mura application's web root directory. So in my main
Mura directory I have a **WEB-INF** folder containing the entire Railo
**library**.

If you open up that **WEB-INF** folder there should be a **web.xml**
file. Open this file in your favorite text editor. Near the top, after
the **<display-name>** node and before the **<servlet>** nodes paste
the following XML.

{% highlight xml %}
<!-- URL Rewrite Filter -->
<filter>
	<filter-name>UrlRewriteFilter</filter-name>
	<filter-class>org.tuckey.web.filters.urlrewrite.UrlRewriteFilter</filter-class>
	<init-param>
		<param-name>logLevel</param-name>
		<param-value>WARN</param-value>
	</init-param>
</filter>
<filter-mapping>
	<filter-name>UrlRewriteFilter</filter-name>
	<servlet-name>CFMLServlet</servlet-name>
</filter-mapping>
{% endhighlight %}

This block of XML tells Tomcat, when loading your application, to
install this URL re-write filter, and to map any requests that come
through the **CFMLServlet** to it. Basically that means that any CFM or
CFC requests will go through the filter.

The next part of this same file that must be altered is the servlet
mappings to allow paths after "index.cfm". This allows Mura to execute
requests like "/index.cfm/contact-us/". To do this find the following
block of XML:

{% highlight xml %}
<servlet-mapping>
	<servlet-name>CFMLServlet</servlet-name>
	<url-pattern>*.cfm</url-pattern>
</servlet-mapping>
{% endhighlight %}

Once you find this, you want to insert this next block of XML **right
after it**.

{% highlight xml %}
<servlet-mapping>
	<servlet-name>CFMLServlet</servlet-name>
	<url-pattern>/index.cfm/*</url-pattern>
</servlet-mapping>
{% endhighlight %}

Now we are ready to configure the re-writer filter to forward our
"pretty" SES URLs to index.cfm! In the extracted files there is a file
named **urlrewrite.xml**. Copy this file into the same folder as your
Railo **web.xml** file that we just got done modifying. Now open it up.
Fortunately this file is pretty well documented, and the online docs
have even more information! For the moment, though, we just want to make
sure that all of our SES URLs go to the right place. The other concern
here is to also ensure that when we go to Mura's administrator, the URLs
**DO NOT** get forwarded to **index.cfm**. So, with a little bit of
regular expression magic, we can make UrlRewriteFilter do this.

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE urlrewrite PUBLIC "-//tuckey.org//DTD UrlRewrite 3.2//EN" "http://tuckey.org/res/dtds/urlrewrite3.2.dtd">

<!--
	Configuration file for UrlRewriteFilter
	http://tuckey.org/urlrewrite/
-->
<urlrewrite>
	<rule>
		<note>Makes all requests forward to /index.cfm/whatever-came-next
		Ignores admin URLs so the admin still works correctly for Mura.</note>
		<from>^/(?!admin|plugins|js|css|assets|images|tasks|railo-context|flex2gateway|wysiwyg)(.*)$</from>
		<to>/index.cfm/$1</to>
	</rule>
</urlrewrite>
{% endhighlight %}

Ok, super. The final phase is to tell Mura that we wish to make our
URL's pretty! To do this, start Tomcat back up, then log into your Mura
administrator and browse to **File Manager -> Application Root** and
click on the folder **config**. In this folder there is a file named
**settings.ini.cfm**. Click on the **pencil** icon to the right of the
file name to edit it. Once inside we are looking for two settings:
**siteidinurls** and **indexfileinurls**. These settings should both be
set to zero (0). Once you save those changes make sure to click on
**Reload Application**.

And with that you should be able to browse to your Mura site **without**
the crazy question marks or **index.cfm** in the URL. But if you are in
the administrator you should still have all the question marks so it
behaves as expected.

I hope this helped somebody out there. Cheers, and happy coding!
