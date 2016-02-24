---
author: "Adam Presley"
date: 2010-12-01T21:06:00Z
tags: ["groovy", "development"]
title: "Enhancing Groovy - SHA1 Hash Strings"
slug: "enhancing-groovy-sha1-hash-strings"
---

Continuing the series of enhancing Groovy's core language classes, here
is a snippet that allows a **String** to be hashed using the SHA1
algorithm. Let's look at the code first.

{{< highlight groovy >}}
import java.security.*

def source = "Hello. This is my secret message. That should be hashed."

String.metaClass.toSHA1 = { salt = "" ->
   def messageDigest = MessageDigest.getInstance("SHA1")

   messageDigest.update(salt.getBytes())
   messageDigest.update(delegate.getBytes())

   /*
    * Why pad up to 40 characters? Because SHA-1 has an output
    * size of 160 bits. Each hexadecimal character is 4-bits.
    * 160 / 4 = 40
    */
   new BigInteger(1, messageDigest.digest()).toString(16).padLeft(40, '0')
}

println "Source = ${source}"
println "SHA1 = ${source.toSHA1('salty')}"
{{< / highlight >}}

This script starts innocently enough with a source string that contains
a message to be hashed. We then extend the **String** class with a new
function named **toSHA1()**. This method uses the **MessageDigest**
class from the **java.security** package to run the source string
through the SHA1 hash routine. Also notice the optional **salt**
argument. A salt is used to make the hash a little more secure and can
complicate various hack attacks
(<http://en.wikipedia.org/wiki/Salt_%28cryptography%29>). We then pass
the byte array of the hashed string, as returned by
**messageDigest.digest()**, into the constructor of a **BigInteger**,
and in turn translate that into a string of Base 16 digits. Also notice
the **padLeft**. That makes sure that we have 40 digits, each
hexadecimal character being 4-bits, giving us 160-bits. Check Wikipedia
and you'll see that the SHA1 algorithm requires an output of 160-bits.

This technique could be easily applied to any other hashing routine such
as MD5. Happy coding, and happy extending Groovy!
