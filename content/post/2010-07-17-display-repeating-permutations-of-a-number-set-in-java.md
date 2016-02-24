---
author: "Adam Presley"
date: 2010-07-17T22:31:00Z
tags: ["fun", "development", "java"]
title: "Display Repeating Permutations of a Number Set in Java"
slug: "display-repeating-permutations-of-a-number-set-in-java"
---

At home my daughter has a little "spy" safe where she can keep money,
rocks, or whatever else she wants in there. This safe is guarded by a
passcode, and when you enter the passcode incorrectly twice it makes an
alarm sound. Real cute.

I am always stopping by when she's trying to do something and guess the
passcode, annoying her with the alarm sound. Last night it occurred to
me to write a quick little program to display all possible combinations
for the passcode. So I did. With digits 1, 2, 3, and 4 there are 256
possible combinations.

{{< highlight java >}}
package com.adampresley.NumDigits;

public class Main
{
   public static void main(String[] args) {
      go("1234", 4, new StringBuffer());
   }

   static void go(String input, int depth, StringBuffer output) {
      if (depth == 0) {
         System.out.println(output);
      }
      else {
         for (int i = 0; i < input.length(); i++) {
            output.append(input.charAt(i));
            go(input, depth - 1, output);
            output.deleteCharAt(output.length() - 1);
         }
      }
   }
}
{{< / highlight >}}

After 15 tries my fiancee gets tired of the noise it's making and has my
daughter come take the safe away from me. My daughter then tells me the
actual combination. Social engineering at it's finest!

Happy coding!
