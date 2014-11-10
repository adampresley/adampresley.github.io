---
layout: post
title: Generate a Random String in Python
date: 2012-11-20 21:38:00
author: Adam Presley
status: Published
tags: development python
slug: generate-a-random-string-in-python
---

Here's a tidbit I picked up from Stack Overflow
(<http://stackoverflow.com/questions/2823316/generate-a-random-letter-in-python>)
that I found helpful. I found the need to generate a random string of
digits, but didn't want to just loop and concatenate. I was pretty sure
there was a more *Pythonic* way to do this, and turns out I was right.
I'm going to first show the solution, then we'll break it down to
explain what each little piece is doing.

    :::python
    "".join(random.choice(string.ascii_letters) for i in range(8))

So the above may seem a bit anti-climatic but it demonstrates the raw
expressiveness of the Python language. So I'd like to break this down a
bit to explain the parts.

![Ascii Letters](http://s3.amazonaws.com/www.adampresley.com/posts/python-string-ascii-letters.png)

In the above screenshot you will see the code **string.ascii_letters**
produces both a lower and uppercase alphabet in a single string. This
will be the source from which we will pull letters from to produce our
random string.

![Random letters](http://s3.amazonaws.com/www.adampresley.com/posts/python-random-choice.png)

In this screenshot we are using **random.choice()**. This method will
select a value from a string at random. So when we pass our alphabet
**random.choice()** will pick a letter from it at random.

![Join](http://s3.amazonaws.com/www.adampresley.com/posts/python-string-join.png)

Now we get to see the string's **join()** method in action. The syntax
may seem backwards, but try reading it like this: *Using a **comma**
concatenate each of the characters in the string **12345**. As you can
see the result is a comma-delimited number set.

![8-digits](http://s3.amazonaws.com/www.adampresley.com/posts/python-random-string-8-digits.png)

This final screenshot shows the whole thing in action. We want to join
using blank (so there will no delimiter) a random choice from the
alphabet, and do this eight (8) times. The loop **for i in range(8)**
does the actual loop using a *loop comprehension*.

I hope this was mildy informative and useful. Cheers, and happy coding!