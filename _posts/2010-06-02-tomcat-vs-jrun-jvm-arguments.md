---
layout: post
title: "Tomcat vs JRun: JVM Arguments"
date: 2010-06-02 12:45:00
author: Adam Presley
status: Published
tags: development tomcat jee
slug: tomcat-vs-jrun-jvm-arguments
---
Lately I have been learning more about running ColdFusion applications
in JEE application servers and servlet containers. The product I have
focused on primarily is [Apache's Tomcat](http://tomcat.apache.org/), a
very free, open source, and scalable JEE platform. So far I have enjoyed
the experience.

However recently I have been asked to do some performance testing on
various portions of the application I work on for my day job. And today
I came across disturbing results. JRun was actually performing
**faster** than Tomcat! Eeep! This seemed to me a travesty
considering the age and lack of support for the JRun product.

So originally my JVM arguments looked like this.

{% highlight text %}
set JAVA_OPTS=-Xms256m -Xmx512m -XX:+UseConcMarkSweepGC -XX:GCTimeRatio=19
{% endhighlight %}

Basically this is telling Tomcat to run with a minimum heap size of
256MB, a maximum heap size of 512MB, use the concurrent sweep garbage
collector. The *-XX:GCTimeRatio* argument tells the JVM to have a goal
of spending 5% of its time on garbage collection, the remaining being
devoted to application time. With this configuration I was finding JRun
to be the faster server. Why??

Then I noticed a little parameter in my JRun JVM configuration,
**-server**. After a little searching I have found that the HotSpot
JVM has two modes of operation: **-client** and **-server**. The
difference appears to be how the memory manager and compiler optimizes
memory and class compiling/instantiation. For the client configuration
it apparently optimizes speed more toward new generation objects, while
the server configuration flag optimized for better perm generation
management.

For the application I was working on this made a big difference! Adding
that flag to my JVM arguments enhanced performance in Tomcat for this
app, and made me much happier! So now my JVM argument setting in
*catalina.bat* looks like this.

{% highlight text %}
set JAVA_OPTS=-server -Xms256m -Xmx512m -XX:+UseConcMarkSweepGC -XX:GCTimeRatio=19
{% endhighlight %}
