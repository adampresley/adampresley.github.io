---
author: "Adam Presley"
date: 2010-12-02T23:27:00Z
tags: ["development", "javascript"]
title: "Firebug's Console - Making Sure We Don't Break IE"
slug: "firebugs-console-making-sure-we-dont-break-ie"
---

Here's a quick little JavaScript function that allows you to continue
your loving relationship with Firebug in Firefox, but also not break
your Internet Explorer users. This function wraps the **console.log()**
function and makes sure that it exists before it tries to use it. This
avoids causing JavaScript errors in IE.

{{< highlight javascript >}}
log = function(msg) {
    if ("console" in window) {
        console.log(msg);
    }
};
{{< / highlight >}}

To test this create a page with that code that looks something like the
code below. Then run it in Firefox and Internet Explorer.

{{< highlight javascript >}}
log = function(msg) {
   if ("console" in window) {
      console.log(msg);
   }
};

log("hi!");
{{< / highlight >}}

Short, but useful, and this can clearly be extended to do many more
useful things. It can also easily be changed to check for Firebug Lite
in IE, or even just drop down to an **alert** if necessary. Happy
coding!
