---
author: "Adam Presley"
date: 2011-03-29T05:47:00Z
tags: ["development", "csharp"]
title: "MD5 Hash a String in C#"
slug: "md5-hash-a-string-in-c"
---

Here's a small function to hash a string in C# using the MD5 algorithm.

{{< highlight csharp >}}
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
{{< / highlight >}}

Note that the **Replace()** at the end is optional and you may remove it
should you want the dashes in your hash string.

Happy coding!
