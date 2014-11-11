---
layout: post
title: Setting a Proxy for the Grails Command Line Tools
date: 2010-11-23 21:54:00
author: Adam Presley
status: Published
tags: development grails
slug: setting-a-proxy-for-the-grails-command-line-tools
---

If you've ever worked at a company that uses a corporate proxy you may
have run into this little issue. Some command line tools such as CURL or
Grails make calls out to the internet to do their jobs, and some
corporate proxies just get in the way. Grails has a way of handling
this, however, and you can setup a proxy so that Grails commands execute
successfully.  
  
{% highlight bash %}
$ grails add-proxy client --host=1.2.3.4 --port=80 --username=myUser --password=passwordgrails set-proxy client
{% endhighlight %}

In the above example I am setting up a proxy configuration with a name
of **client**. I then tell Grails to use that named proxy by using the
**set-proxy** command. There is a catch, however, if you are stuck in
Windows. In my Windows 7 64-bit machine environment I ran into a problem
where the proxy didn't seem to setup correctly. To remedy this browse to
your user folder and open up the **.grails/ProxySettings.groovy** file.
In there you will see your named proxy, **client** in my case, and you
will notice that all the values are wrong. Correct those values by
making them strings and save the changes. Note that by strings I mean
they must have quotes around the values (after the equal sign).   
  
Then try your Grails commands again and you should be golden! Happy
coding!
