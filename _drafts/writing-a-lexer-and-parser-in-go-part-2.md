---
layout: post
title: Writing a Lexer and Parser in Go - Part 2
date: 2015-04-12 22:48
author: Adam Presley
tags: development golang
comments: true
---
In [part one]({% post_url 2015-04-12-writing-a-lexer-and-parser-in-go-part-1 %}) of this series I introduced the concepts around lexical analysis and parsing. We took a look at the basics behind what makes up an INI file, and started setting up structures and constants that will help us perform lexical analysis, or *lexing*, on an input text, which is an INI file in our case.

In part two we actually dive into the details of the process of performing a lexical analysis. Lexing is the process of converting an input text into a series of tokens. Token represent smaller units of the input that, when put together, become something useful, like a program, or configuration file, and more. In an INI file token include left bracket, right bracket, section name, key name, value, and equal sign. Put together in the correct order and you have an INI file. The job of the lexer is to read the INI file and create tokens, placing them onto a channel for a parser to pick up and do something useful with.

