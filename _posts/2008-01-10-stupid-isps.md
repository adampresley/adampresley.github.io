---
layout: post
title: Stupid ISP's
date: 2008-01-10 16:22:00
author: Adam Presley
status: Published
tags: development
slug: stupid-isps
---

For several days now I have been fighting with my computer to get Apache
configured so that I can access the site I'm working on, not only from
the inside, but also from the outside for demo purposes. I already knew
about the UAS stuff everybody warns you about with Vista, and I already
had all of my virtual host entries setup. Still no love. Port
forwarding? Sure thing, and still no love.

After a little more obscure searching I found information that stated
that some ISP's will block incoming port 80 traffic requests. Worth a
shot I figured, so I setup the router to forward 8080. Got Apache
configured. Started everything back up, and viola! Love!
