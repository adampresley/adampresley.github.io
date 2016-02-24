---
author: "Adam Presley"
date: 2011-06-11T16:44:00Z
tags: ["development"]
title: "GAE and Client Error 400"
slug: "gae-and-client-error-400"
---

Ok, this one was a beating. For my wedding website I am using Google App
Engine, a cloud PaaS system. The application is written in [Groovy](http://groovy.codehaus.org/) on
[Gaelyk](http://gaelyk.appspot.com/). So far everything has been great, until yesterday.

While attempting to deploy my application I started getting an error
like this:

{{< highlight java >}}
Unable to update:java.io.IOException:
Error posting to URL: https://appengine.google.com/api/appversion/deploy?app_id=adammaryannewedding&version=4&400 Bad
RequestClient Error (400)
The request is invalid for an unspecified reason.
{{< / highlight >}}

This infuriating error message is so maddening because it tells you
absolutely nothing about what could be going wrong. After a **lot** of
searching I found some old forum entries talking about getting this
error if you have too many files to upload. How many is too many? Turns
out that the hard limit is 1,000 files.

I understand limits. Even though I think 1,000 is a bit light, throw me
a bone and at least let me **KNOW** that this was the problem!

Oh, and feel free to visit my wedding site at
[www.adamandmaryannewedding.com](http://www.adamandmaryannewedding.com). Cheers!
