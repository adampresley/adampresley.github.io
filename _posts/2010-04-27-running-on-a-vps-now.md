---
layout: post
title: Running on a VPS Now
date: 2010-04-27 21:25:00
author: Adam Presley
status: Published
tags: technology general server
slug: running-on-a-vps-now
---
About a week ago I moved all the code for my site onto a new VPS
(Virtual Private Server) provided by [2Host.com](http://www.2host.com).
With their unmanaged service I get 1GB of RAM, 30GB of hard drive space,
and over 9TB of bandwidth, and I get it at a great price! So far the
service has been very stable, with only a few issues with the newest
Ubuntu VM images.

Note that setting up your own Linux VPS is not for the faint of heart,
and should only be attempted by the more nerdy. :) Anyway, I got Apache
2 and PHP 5 setup, along with MySQL 5, Tomcat 6, and Railo running my
Coldfusion applications. I must say that it is *FAST*, and I am most
pleased with that.

I did run into initial setup issue when I tried using the Ubuntu 9.10
image. MySQL would install, but put no default *root* user for me to
administer with. The fix ended up being me going back to Ubuntu 8.04,
and those problems seemed to disappear.

Overall this has been a good experience, and I hope it continues to be
so! Cheers!
