---
layout: post
title: MD5 Hash a String in C#
date: 2011-03-29 05:47:00
author: Adam Presley
status: Published
tags: development c#
slug: md5-hash-a-string-in-c
---

Here's a small function to hash a string in C# using the MD5 algorithm.

{% highlight csharp %}
using System;
using System.Text;
using System.Security.Cryptography;

public String md5Hash(String input) {
    String result = "";

    using (MD5 md5 = new MD5CryptoServiceProvider()) {
        result = BitConverter.ToString(md5.ComputeHash(ASCIIEncoding.Default.GetBytes(input)));
    }

    return result.Replace("-", String.Empty);
}
{% endhighlight %}

Note that the **Replace()** at the end is optional and you may remove it
should you want the dashes in your hash string.

Happy coding!
