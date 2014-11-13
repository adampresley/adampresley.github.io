---
layout: post
title: Update v0.2.0 to jQuery Plugin - AP Tags
date: 2009-10-19 18:34:00
author: Adam Presley
status: Published
tags: development jquery javascript
slug: update-v020-to-jquery-plugin-ap-tags
---
While using my little tag plugin, [AP Tags](http://plugins.jquery.com/project/aptags), I ran into a small snag
where I have a dialog box that I am crafting dynamically using [jQuery
UI](http://jqueryui.com/). In this dialog I have my tag plugin, and I am calling it every
time a different "Edit" link is created. I'm doing this so I can
populate the tag editor with the tags of whatever item I'm editing.

It would seem that every time I call AP Tags, however, new spans and
hidden field DOM elements are being created, causing duplicate DOM
elements to be created. I have corrected this in version 0.2.0, and it
is now available on the [jQuery website](http://plugins.jquery.com/project/aptags) for download.

Cheers, and happy coding!
