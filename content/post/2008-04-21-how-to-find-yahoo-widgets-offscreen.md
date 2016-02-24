---
author: "Adam Presley"
date: 2008-04-21T08:04:00Z
tags: ["general", "nerdy"]
title: "How to Find Yahoo Widgets Offscreen"
slug: "how-to-find-yahoo-widgets-offscreen"
---

For those that enjoy the features offered by the [Yahoo! Widgets](http://widgets.yahoo.com) you
might at some point find yourself in a situation similar to what I
experienced today. Somehow, perhaps with my two monitor goodness, and
goofing that up by logging into our VPN, I tried to open up the
[Converter widget](http://widgets.yahoo.com/widgets/widget-converter), and it never showed up on the screen.

After a bit of digging I at least found out how to reset the widget's
position, even though I didn't find out why it disappeared. If you find
that launching a widget gives you a non-visible widget, you may wish to
try the following.

1. Open Regedit (Start -> Run -> Regedit)
* Browse to HKEY_CURRENT_USER\Software\Yahoo\WidgetEngine
* Find the folder with the name of the widget. Under this folder will likely be a folder named Windows.
* Under the Windows folder there should be a main folder. Open this.
* Delete the key named Positions
* Restart the widget

Good widgeting!
