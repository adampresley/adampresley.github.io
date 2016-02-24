---
author: "Adam Presley"
date: 2010-08-19T08:45:00Z
tags: ["development", "tools", "eclipse"]
title: "Change Default Author Name for JavaDocs in Eclipse"
slug: "change-default-author-name-for-javadocs-in-eclipse"
---

Here is a quick way to change the default author name when using JavaDoc
comments in your Eclipse projects. Simply edit your **eclipse.ini**
file found in the root directory where you placed Eclipse. I have
Eclipse at **/opt/eclipse**, so my path would be
**/opt/eclipse/eclipse.ini**. Once editing this file add the
following line and save.

{{< highlight ini >}}
-Duser.name=Adam Presley
{{< / highlight >}}

After saving restart Eclipse and when you do a JavaDoc comment and use
the author attribute by typing **@author** and pressing enter on the
autocomplete you will see something like this:

{{< highlight javascript >}}
/**
 * Alert some message!
 * @author Adam Presley
 */
function alertSomething(msg) { alert(msg); }
{{< / highlight >}}

Simple yet useful. Happy coding!
