---
layout: post
title: Writing a Lexer and Parser in Go - Part 2
date: 2015-04-12 22:48
author: Adam Presley
tags: development golang
comments: true
---
In [part one]({% post_url 2015-04-12-writing-a-lexer-and-parser-in-go-part-1 %}) of this series I introduced the concepts around lexical analysis and parsing. We took a look at the basics behind what makes up an INI file, and started setting up structures and constants that will help us perform lexical analysis, or *lexing*, on an input text, which is an INI file in our case.

In part two we actually dive into the details of the process of performing a lexical analysis. Lexing is the process of converting an input text into a series of tokens. Tokens represent smaller units of the input that, when put together, become something useful, like a program, or configuration file, and more. In an INI file tokens include left bracket, right bracket, section name, key name, value, and equal sign. Put together in the correct order and you have an INI file. The job of the lexer is to read the INI file and create tokens, placing them onto a channel for a parser to pick up and do something useful with.

<!-- excerpt -->

## The Lexer

So let's jump right in. The lexer is the component that converts text input to tokens. To do this we need to track a few things. First we need to keep track of the input text, our current position in that text, and a start and end position of the current token we are potentially looking at. Then we need a a channel to write these tokens to. The final piece is a function representing the current *state* of our lexer. In [Rob's Presentation](http://cuddle.googlecode.com/hg/talk/lex.html#landing-slide) he talks about using functions to track the current state and next expected state of the lexer. This is done by calling a function to process the text, turn it into a token, then return the next lexer function that will process the next expected token.

For example, a section in an INI file consists of a left bracket, followed by a section name, then a right bracket. The first function processes a left bracket, then returns a function that can process a section name. When that function is called it gets the token, then returns the function to process a right bracket. And so on... ```LexLeftBracket -> LexSection -> LexRightBracket```

Let's take a look at that lexer structure. Take notice that we have types defined for a **Token**, which we defined in part 1, and a **LexFn**, which is the lexer state functions that process tokens.

{% highlight go %}
type Lexer struct {
  Name   string
  Input  string
  Tokens chan lexertoken.Token
  State  LexFn

  Start int
  Pos   int
  Width int
}
{% endhighlight %}
