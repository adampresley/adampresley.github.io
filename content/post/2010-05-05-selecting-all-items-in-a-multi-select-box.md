---
author: "Adam Presley"
date: 2010-05-05T11:56:00Z
tags: ["development", "jquery", "javascript"]
title: "Selecting All Items in a Multi-Select Box"
slug: "selecting-all-items-in-a-multi-select-box"
---

Here's a quick [jQuery](http://jquery.com) tidbit. If you ever
need to select or unselect all items in a muti-select box
(a \<select\> tag set to allow multiple selected items), here's
a jQuery selector to do it.

{{< highlight javascript >}}
$("#selectBox *").attr("selected", "selected");
{{< / highlight >}}

Notice the asterisk after the ID selector? That tells jQuery to get
*ALL* child elements of the select box. We then set the **selected**
attribute to **selected**, and suddenly all the items are highlighted!
To unselect them all you would do the opposite.

{{< highlight javascript >}}
$("#selectBox *").attr("selected", "");
{{< / highlight >}}

Setting the attribute **selected** to blank unselects them all. Man I
love jQuery!!

Happy coding!
