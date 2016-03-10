---
author: "Adam Presley"
comments: false
date: 2016-03-10T00:29:00Z
tags: ["development"]
title: "MailSlurper 1.9 Released"
slug: "mailslurper-1-9-released"
---

I am pleased to announce the 1.9 release of MailSlurper. There are quite a few changes in this release.

### UI
* When viewing an email the row you click on is highlighted
* Address regression where searching the **From** field failed to work
* Added ability to sort the email list view ascending or descending by date, subject, or from

### Server
* Added a new option to **config.json** to auto-open your browser when MailSlurper starts. It defaults to turned off
* When no worker is available to service an incoming mail connection the server pool will timeout after 2 seconds and close the connection
* Fixed dates in the format of **2 Jan 2006 15:04:05 -0700**. Found this while testing from a C# client using the *SmtpClient* class
* Server now support SSL by providing certificate and key files. See [the documentation](https://github.com/mailslurper/mailslurper/wiki/Server-Configuration) for more details

Visit the [release page](https://github.com/mailslurper/mailslurper/releases/tag/release-1.9) to download a fresh copy. Please note that the **config.json** file has two new changes, and you should make a backup of your existing file to merge over any customizations to the new file. Cheers, and happy slurping!