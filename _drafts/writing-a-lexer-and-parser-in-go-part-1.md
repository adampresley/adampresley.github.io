---
layout: post
title: Writing a Lexer and Parser in Go - Part 1
date: 2015-04-08 20:35
author: Adam Presley
tags: development golang
comments: true
---
In this three-part series I will talk about building a simple lexer and parser in [Go](http://golang.org). The work presented here is heavily based on a 2011 presentation by Rob Pike titled [Lexical Scanning in Go](http://cuddle.googlecode.com/hg/talk/lex.html#landing-slide). This series will conclude with a fully functional set of code that can parse INI files. The series breaks down into these three parts.

1. Introduction to Lexing, Parsing, and Getting Started
2. Performing Lexical Analysis
3. Parsing the Results

<!-- excerpt -->

## Introduction
Lexical analysis and parsing sound like complicated topics. They sound like complicated topics because they are! This does not mean that we can't break them down a little and grasp the basics, which I will attempt to do here in this three-part series. To do this we will build a simple, functional INI parser. This parser will read an input string of text and return a structure broken into arrays of sections and key/value pairs. We will write this in Go.

INI files have been chosen due to their simple, easy to understand structure. In case you need a refresher an INI file has the following form.

{% highlight ini %}
[SectionName]
key1=value 1
key2=value 2
{% endhighlight %}

In the above sample we see three primary elements. Understanding these elements becomes critical to understand as we start to design our parser.

* Sections
* Keys
* Values

Keys have values that are organized into sections. There may be more than one key/value pair per section, and there may be one or more sections per INI file. It is a simple, but effective structure for storing information such as settings and configuration. We will take this structure and parse it, coming out with structures of data. This JSON structure provides a visual of what our data structures will look like.

{% highlight javascript %}
{
  "FileName": "sample.ini",
  "Sections": [
    {
      "Name": "SectionName",
      "KeyValuePairs": [
        {
          "Key": "key1",
          "Value": "value 1"
        },
        {
          "Key": "key2",
          "Value": "value 2"
        }
      ]
    }
  ]
}
{% endhighlight %}

## What is Lexical Analysis?
Lexical Analysis, or *lexing*, is defined by [Wikipedia](http://en.wikipedia.org/wiki/Lexical_analysis) as *"...the process of converting characters into a sequence of tokens, i.e. meaningful character strings."* Lexing is often executed prior to executing or compiling code. For example, [PHP](http://php.net) is an interpreted language. This means that when you vist a website that runs on PHP code the PHP interpreter runs the PHP code and sends the resulting HTML markup back to your browser. The PHP code first goes through a lexical analysis pass that turns the code into a sequence of tokens that are meaningful to the PHP interpreter. The interpreter then can do something meaningful with these tokens, such as cache for later, execution, etc...

## What is a Token?
A *token* is a structure that describes and categorizes an element from your text input. From the example INI file above the section could be categorized as **TOKEN\_SECTION**, or keys categorized as **TOKEN\_KEY**. This structure generally tracks the category of the element, and the value. As an example our section in the example is named ```SectionName``, and could be stored in a token like the structure below.

{% highlight javascript %}
{
  "Type": TOKEN_SECTION,
  "Value": "SectionName"
}
{% endhighlight %}

With this a parser, interpreter or compiler can make decisions, execute code, or even generate data/code based on what is found in the token structure.

## What is Parsing?
A lexer breaks down the structure of a text input and returns streams of tokens. Having tokens by themselves, however, offers little value. This is where a parser comes in. *Parsing* is the process of performing a syntactic analysis on the stream of tokens provided by the lexer. This means that the parser ensures the input text form is valid and makes sense. For example, the sample below is not a valid INI section header.

{% highlight ini %}
[SectionName]=Hi there
{% endhighlight %}

The lexer may return a series of tokens representing the section name, equal sign, and the string. This is ok, as that is what a lexer does. The parser's job is to determine if that *makes sense*. For an INI file, it does not. Our parser will take a stream of tokens from a Go channel and create data structures containing sections and key/value pairs.

## Breaking it Down
As we near

## What's Next?
