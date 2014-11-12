---
layout: post
title: MailSlurper 3.0 Released
date: 2014-05-05 05:23:00
author: Adam Presley
status: Published
tags: development software golang
slug: mailslurper-3-released
---

I am proud to announce the 3.0 release of MailSlurper, the handy local development mail server that slurps mail into oblivion. When writing application that send mail I use MailSlurper to capture outgoing mail to ensure mail is working, and the actual mail item is what I expect it to be. MailSlurper is written in Google's Go language, with an administrator written in lots of JavaScript. This release includes:

* Mails that contain HTML or are multipart text and HTML now display HTML in the viewer
* Added ability to search the subject and bodies of mails to filter mail list
* Added sorting of mail items
* Addressed a date parsing issue with mails that have the timezone wrapped in parentheses
* Addressed browser resize issue. Layout now is resizable and more responsive
* Removed unneeded code
* Updated several libraries

You can get the new release at [the Github site](https://github.com/adampresley/mailslurper-go/releases/tag/3.0). 

![MailSlurper 3.0](/assets/adampresley/images/posts/mailslurper-3.0.png)