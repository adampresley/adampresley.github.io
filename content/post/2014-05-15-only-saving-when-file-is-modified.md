---
author: "Adam Presley"
date: 2014-05-15T13:48:00Z
tags: ["development", "python", "sublime"]
title: "Only Saving When File Is Modified"
slug: "only-saving-when-file-is-modified"
---

I got a ticket the other day questioning why my Sublime Text [View In Browser](https://github.com/adampresley/sublime-view-in-browser) plugin saves the file they are viewing every time the plugin is used. The answer of course is because it must be saved prior to opening in the browser. However the poster of the ticket did have a good point. There is no need to save the user's file if the file hasn't actually changed. I have pushed a modification to the plugin to ensure that the save only occurs if the user's file has any modifications.

To do this in Sublime's API turned out to be very simple.

{{< highlight python >}}
if self.view.is_dirty():
    self.view.window().run_command("save")
{{< / highlight >}}

The *view* object gives my plugin insight into the current view, or file being worked on. Sublime provides a nice method named **is_dirty()** which will tell me if the current view has any pending modifications. If it does I perform the save command. If not the rest of the plugin runs and your file is opened in your browser of choice.