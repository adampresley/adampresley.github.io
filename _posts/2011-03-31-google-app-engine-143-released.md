---
layout: post
title: Google App Engine 1.4.3 Released
date: 2011-03-31 11:23:00
author: Adam Presley
status: Published
tags: development google
slug: google-app-engine-143-released
---

I'm about a day late to the game, but Google has released version 1.4.3
of their cloud-based PaaS. For those of us using [OpenBD](http://openbd.org) on the
Google App Engine the latest enhancements to the Java platform side are
particularly welcome.

* Concurrent Requests - This one is a big #winning! Instead of firing up more app instances, a single app instance can handle more requests. It does require a configuration change in your application.
* Java Remote API - This provides the ability to execute actions against your datastore remotely. Seeing that I am crafting webservices for this kind of thing right now, remote access could be a big deal. We'll see.
* Files API - About...time... Ability finally to be able to do file-like manipulation with their Blobstore. Let's hope it is as awesome as it sounds.

Although not a **huge** release, there are some pretty big items in
there that could have dramatic effect your applications. The official
release notes are [here](http://googleappengine.blogspot.com/2011/03/announcing-app-engine-143-release_30.html).
