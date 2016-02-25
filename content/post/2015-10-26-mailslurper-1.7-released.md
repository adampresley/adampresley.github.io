---
author: "Adam Presley"
comments: false
date: 2015-10-26T00:00:00Z
tags: ["development"]
title: "MailSlurper 1.7 Released"
slug: "mailslurper-1-7-released"
---

MailSlurper 1.7 has been released. This version has a couple of bug fixes and a couple of new features. I also reworked a bit of the back-end server worker pool for better concurrency. Below are the release notes. Visit [MailSlurper.com](http://mailslurper.com) to download now.

* Fixed typo in server log messages
* Previously a client connection that sent incomplete attachment information would cause MailSlurper to panic. MailSlurper has grown some courage in this release, and no longer panics
* Users may now choose between two date formats: US (MM/DD/YYYY) and International (YYYY-MM-DD)
* There is now an option to have your list of mail items auto refresh on an interval
