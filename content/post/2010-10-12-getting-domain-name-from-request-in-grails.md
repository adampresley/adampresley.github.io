---
author: "Adam Presley"
date: 2010-10-12T05:10:00Z
tags: ["development", "groovy", "grails"]
title: "Getting Domain Name From Request in Grails"
slug: "getting-domain-name-from-request-in-grails"
---

Here's a little tidbit. If you need to determine what domain a request
may have been referred from, such as when doing JSONP work, here's a
code snippet that might help.

{{< highlight groovy >}}
def uri = new java.net.URI(request.getHeader("referer"))
def domainName = uri.getHost()
{{< / highlight >}}

The *java.net* package contains the **URI** class which aides in
working with URIs and URLs. And in Grails the request processed by the
servlet engine is an object named **request**. That object contains
a method called **getHeader** that allows you to get any header
value by name.

Using this little bit of code I am able to check the referring domain of
a JSONP request against a MongoDB collection that contains a list of
valid domains for a given registered user. Then if the request came from
a valid domain I serve up the requested JSON response.

Hope that's helpful to someone. Happy coding!
