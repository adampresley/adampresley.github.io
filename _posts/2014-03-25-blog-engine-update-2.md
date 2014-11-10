---
layout: post
title: Blog Engine - Update 2
date: 2014-03-25 05:43:00
author: Adam Presley
status: Published
tags: development programming python
slug: blog-engine-update-2
---

Back in November 2013 I released my blog site under a new engine written by me. Before that I had run my site on Blogger for a while, but I grew tired of how sluggish it had become. I felt I could get better performance. So I set out to write my own using Python and lots of JavaScript.

Initially I wrote this with my site being fully AJAX driven. When you hit a URL you were hitting URI fragments which would hit a JavaScript controller I had written to route you to the correct blog entry or page of entries. This worked fairly well performance wise, but suffered from one major problem that I was ignorant of. *It became a giant pain in the butt to index my site with Google*. After researching the issue I did find that Google offers a solution for indexing AJAX-heavy sites, but it required me to come up with clever, convoluted solutions to create static renders of each blog post and page of posts. This turned out to be more trouble than it's really worth.

I decided to refactor a bit, though it has been a slow process. I also wanted to expand on the administrator I had built for myself and turn this into a proper CMS engine, or at least start down that path. Tonight I have released the newest increment of my site using the latest version of my code. I am calling my CMS engine **Texo CMS**. Of course it only really does blog posts right now, and doesn't really deserve the title of **CMS**, but I'm working on it.

![Screenshot](http://www.adampresley.com.s3.amazonaws.com/posts/texo-cms-write-post-1.png)

My next steps are to continue development on this application, and make a home for it on Github. More on that soon.