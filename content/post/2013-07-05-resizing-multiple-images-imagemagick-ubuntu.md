---
author: "Adam Presley"
date: 2013-07-05T11:52:00Z
tags: ["general", "tools"]
title: "Resizing Multiple Images Using ImageMagick in Ubuntu"
slug: "resizing-multiple-images-imagemagick-ubuntu"
---

Have you ever needed to resize a bunch of images in bulk? Most of us who do web development,
image manipulation, or blogging probably have at some point. If you are running in Ubuntu here
is a quick way to do this on the console.

{{< highlight bash >}}
$ convert '*.JPG[800x>]' new-name-%03d.jpg
{{< / highlight >}}

The above script will take all files with a **.JPG** extension and resize them to 800 pixels
wide and name the new version of the file as *new-name-001.jpg*. The **%03d** tells
ImageMagick to add numbers sequentially using 3 digits. Pretty cool eh?
