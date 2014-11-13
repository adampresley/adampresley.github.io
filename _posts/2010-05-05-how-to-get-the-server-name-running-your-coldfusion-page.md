---
layout: post
title: How To Get the Server Name Running Your ColdFusion Page
date: 2010-05-05 05:46:00
author: Adam Presley
status: Published
tags: development coldfusion java
slug: how-to-get-the-server-name-running-your-coldfusion-page
---
ColdFusion is a great language in its own right. What makes it even more
awesome? The fact that it is a Java application. Why is this cool?
Because you can dig into Java to do all the dirty stuff ColdFusion
shields you from, should you desire to do so.  
  
While writing this profiling code this last week I had the need to get
information specific to the server executing my ColdFusion code. The
first bit of information I need is the host name, or the proper name of
the server running ColdFusion. Java to the rescue! The Java package
**java.net** contains what we need to do this.  

{% highlight cfm %}
<cfset inet = createObject("java", "java.net.InetAddress") />
<cfset hostName = inet.getLocalHost().getHostName() />

<cfoutput>This computer's name is #hostName#</cfoutput>
{% endhighlight %}

The other piece of information I needed was something to identify the
*instance* of ColdFusion I was running on in a multi-instance
environment. I'm betting the way I did this could be done better, but
for now it is doing the job. By default ColdFusion installs JRun as the
J2EE server that runs ColdFusion as a Java application. As such I was
able to delve into the Java classes provided by JRun to get the JNDI
port number, which is of course unique per instance.  
  
The code below shows how to do that. Also note I have a try/catch around
that code, and in the catch I am trying to create a different object of
type **org.apache.catalina.ServerFactory**. This is because my
co-workers are running JRun, and I am running Tomcat, and that object is
what get's me a server class from Tomcat, which in turn can give me the
port Tomcat listens on for the shutdown command.  

{% highlight cfm %}
<!---
    Get the port number. This is *J2EE server specific*!
--->
<cftry>
    <cfset j2eeService = createObject("java", "jrun.naming.NamingService") />

<cfcatch>
    <cfset j2eeService = createObject("java", "org.apache.catalina.ServerFactory").getServer() />
</cfcatch>
</cftry>

<cfset port = j2eeService.getPort() />

<cfoutput>This instance is running against port #port#</cfoutput>
{% endhighlight %}

And this, ladies and gentlemen, is why ColdFusion rocks! Happy coding!
