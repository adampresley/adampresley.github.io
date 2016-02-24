---
author: "Adam Presley"
date: 2010-06-08T14:45:00Z
tags: ["development", "ant"]
title: "'Update: ANT to Automate Build Version Number'"
slug: "update-ant-to-automate-build-version-number"
---

In a previous post [Using ANT to Automate a Build Version Number](#post/2009/01/using-ant-to-automate-a-build)
I talk about incrementing build numbers using ANT. I found this
particularly useful when building Mura plugins. Thanks to [Steve Good](http://stevegood.org)
for pointing out a little issue where if you wish to use the build
number in naming your ZIP file, my example is incomplete. So to pull in
the build number as a property so that you can include it in the ZIP
file's name, you would need to tell ANT to load the property file and
read in the properties like so.

{{< highlight xml >}}
<property file="build.properties" />
<zip destfile="${sourceDir}\HelloPlugin-${buildNumber}.zip" basedir="${buildDir}" />
{{< / highlight >}}

The **${buildNumber}** portion gets the value from the build.properties
file and uses it as part of the filename. Good catch Steve!
