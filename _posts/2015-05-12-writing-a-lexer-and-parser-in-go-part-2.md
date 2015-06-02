---
layout: post
title: Writing a Lexer and Parser in Go - Part 2
date: 2015-05-12 22:48
author: Adam Presley
tags: development golang
comments: true
---
In [part one]({% post_url 2015-04-12-writing-a-lexer-and-parser-in-go-part-1 %}) of this series I introduced the concepts around lexical analysis and parsing. We took a look at the basics behind what makes up an INI file, and started setting up structures and constants that will help us perform lexical analysis, or *lexing*, on an input text, which is an INI file in our case.

In part two we actually dive into the details of the process of performing a lexical analysis. Lexing is the process of converting an input text into a series of tokens. Tokens represent smaller units of the input that, when put together, become something useful, like a program, or configuration file, and more. In an INI file tokens include left bracket, right bracket, section name, key name, value, and equal sign. Put together in the correct order and you have an INI file. The job of the lexer is to read the INI file and create tokens, placing them onto a channel for a parser to pick up and do something useful with.

<!-- excerpt -->

The Lexer
---------

So let's jump right in. The lexer is the component that converts text input to tokens. To do this we need to track a few things. First we need to keep track of the input text, our current position in that text, and a start and end position of the current token we are potentially looking at. Then we need a a channel to write these tokens to. The final piece is a function representing the current *state* of our lexer. In [Rob's Presentation](http://cuddle.googlecode.com/hg/talk/lex.html#landing-slide) he talks about using functions to track the current state and next expected state of the lexer. This is done by calling a function to process the text, turn it into a token, then return the next lexer function that will process the next expected token.

For example, a section in an INI file consists of a left bracket, followed by a section name, then a right bracket. The first function processes a left bracket, then returns a function that can process a section name. When that function is called it gets the token, then returns the function to process a right bracket. And so on... ```LexLeftBracket -> LexSection -> LexRightBracket```

Let's take a look at that lexer structure. Take notice that we have types defined for a **Token**, which we defined in part 1, and a **LexFn**, which is the lexer state functions that process tokens. We will also see the definitions of the lext state function type **LexFn**.

#### Lexer.go

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

#### LexFn.go

{% highlight go %}
type LexFn func(*Lexer) LexFn
{% endhighlight %}

Now let's add some useful functionality to our lexer. Since a lexer analyzes text input we can use some tools for doing this. In this next snippet we will add to *Lexer.go* the ability to get the next token in the stream, read runes, skip whitespace, and other useful tools. Much of this is simply text processing.

{% highlight go %}
/*
Puts a token onto the token channel. The value of this token is
read from the input based on the current lexer position.
*/
func (this *Lexer) Emit(tokenType lexertoken.TokenType) {
  this.Tokens <- lexertoken.Token{Type: tokenType, Value: this.Input[this.Start:this.Pos]}
  this.Start = this.Pos
}

/*
Increment the position
*/
func (this *Lexer) Inc() {
  this.Pos++
  if this.Pos >= utf8.RuneCountInString(this.Input) {
    this.Emit(lexertoken.TOKEN_EOF)
  }
}

/*
Return a slice of the input from the current lexer position
to the end of the input string.
*/
func (this *Lexer) InputToEnd() string {
  return this.Input[this.Pos:]
}

/*
Skips whitespace until we get something meaningful.
*/
func (this *Lexer) SkipWhitespace() {
  for {
    ch := this.Next()

    if !unicode.IsSpace(ch) {
      this.Dec()
      break
    }

    if ch == lexertoken.EOF {
      this.Emit(lexertoken.TOKEN_EOF)
      break
    }
  }
}
{% endhighlight %}

The most important aspects are reading tokens then emitting them to a channel. The lexer does this using the following steps.

1. Read text input until you've read what constitutes as a specific token. For example, in the lexer state function that processes section names we read characters (runes) until we reach a right square bracket.
1. Emit this text and it's token type as a token onto our token channel
1. Return the next expected state function

Now let's setup a function that will start off the process. This will be the starting point for the parser we write in part 3 of this series. It is important to point out here that we initialize the lexer with a state function that is the first expected token type. This could be a specific character or keyword, but in our case we will use a general-purpose state function named **LexBegin**. The reason we need this is because an INI file can either begin with a section, or it can omit the section entirely and simply have key/value pairs. Our state function starts by checking this.

{% highlight go %}
/*
Start a new lexer with a given input string. This returns the
instance of the lexer and a channel of tokens. Reading this stream
is the way to parse a given input and perform processing.
*/
func BeginLexing(name, input string) *lexer.Lexer {
  l := &lexer.Lexer{
    Name:   name,
    Input:  input,
    State:  lexer.LexBegin,
    Tokens: make(chan lexertoken.Token, 3),
  }

  return l
}
{% endhighlight %}

### Begin
Now let's look at that first state function **LexBegin**. As you can see the first order of business is to skip any whitespace. Whitespace has no meaning in an INI file outside of values or section names, so we skip it. Then we look at the input text stream for a left square bracket. If we have one we want to return the state function to process it, otherwise we start with a key name.

{% highlight go %}
/*
This lexer function starts everything off. It determines if we are
beginning with a key/value assignment or a section.
*/
func LexBegin(lexer *Lexer) LexFn {
  lexer.SkipWhitespace()

  if strings.HasPrefix(lexer.InputToEnd(), lexertoken.LEFT_BRACKET) {
    return LexLeftBracket
  } else {
    return LexKey
  }
}
{% endhighlight %}

### Sections
Let's follow the path of a section first. If you recall a section in an INI file is a name enclosed in left and right square brackets. This allows you to organize key/value pairs into named sections. Notice in the **LexBegin** state function if we find a left bracket we return the state function named **LexLeftBracket**. Let's look at that.

{% highlight go %}
/*
This lexer function emits a TOKEN_LEFT_BRACKET then returns
the lexer for a section header.
*/
func LexLeftBracket(lexer *Lexer) LexFn {
  lexer.Pos += len(lexertoken.LEFT_BRACKET)
  lexer.Emit(lexertoken.TOKEN_LEFT_BRACKET)
  return LexSection
}
{% endhighlight %}

This is great place to start as it is as simple as it gets. The first order of business is to increment the lexer's position tracker by the length of a left square bracket (which should be 1 character). Then we emit a token of type *TOKEN_LEFT_BRACKET*. It is worth noting that the **Emit** function automatically captures the token text from the input stream based on the lexer position tracking. It does this by keeping a start position and current position. When you emit a token it is placed on the channel, then the start position is advanced to the position where the current tracker is at. The final step is to return a state function of what we are expecting next, which is a section name.

{% highlight go %}
/*
This lexer function emits a TOKEN_SECTION with the name of an
INI file section header.
*/
func LexSection(lexer *Lexer) LexFn {
  for {
    if lexer.IsEOF() {
      return lexer.Errorf(errors.LEXER_ERROR_MISSING_RIGHT_BRACKET)
    }

    if strings.HasPrefix(lexer.InputToEnd(), lexertoken.RIGHT_BRACKET) {
      lexer.Emit(lexertoken.TOKEN_SECTION)
      return LexRightBracket
    }

    lexer.Inc()
  }
}
{% endhighlight %}

This state function is a little more complicated, but follows the same premise. This one will loop until we reach a right square bracket, which signifies the end of a section name. Notice how we are looking for EOF? If we reach the end of the input stream then we have a malformed INI file, and we should put an error on the token channel. If all is well, however, we continue looping until we find that bracket, then emit a token of type *TOKEN_SECTION*.

Then we return the state function to process the right square bracket. It is very similar to the left square bracket function, except that when it is done it returns us back to the begin state where we started. This is because you can have sections that are empty and contain no key/value pairs.

{% highlight go %}
/*
This lexer function emits a TOKEN_RIGHT_BRACKET then returns
the lexer for a begin.
*/
func LexRightBracket(lexer *Lexer) LexFn {
  lexer.Pos += len(lexertoken.RIGHT_BRACKET)
  lexer.Emit(lexertoken.TOKEN_RIGHT_BRACKET)
  return LexBegin
}
{% endhighlight %}

### Key/Value Pairs
Next up is the processing of key/value pairs. The format is easy: ```key=Some value here```. To begin we start with the state function which processes keys. Much like the section state function this one will loop until we reach an equal sign, which signifies the end of a key name. When it does it emits a token of type *TOKEN_KEY*, then returns the state function **LexEqualSign**.

{% highlight go %}
/*
This lexer function emits a TOKEN_KEY with the name of an
key that will be assigned a value.
*/
func LexKey(lexer *Lexer) LexFn {
  for {
    if strings.HasPrefix(lexer.InputToEnd(), lexertoken.EQUAL_SIGN) {
      lexer.Emit(lexertoken.TOKEN_KEY)
      return LexEqualSign
    }

    lexer.Inc()

    if lexer.IsEOF() {
      return lexer.Errorf(errors.LEXER_ERROR_UNEXPECTED_EOF)
    }
  }
}
{% endhighlight %}

The equal sign is simple enough. It is very similar to the right and left square bracket state functions. This emits the *TOKEN_EQUAL_SIGN* token type then returns the **LexValue** state function.

{% highlight go %}
/*
This lexer function emits a TOKEN_EQUAL_SIGN then returns
the lexer for value.
*/
func LexEqualSign(lexer *Lexer) LexFn {
  lexer.Pos += len(lexertoken.EQUAL_SIGN)
  lexer.Emit(lexertoken.TOKEN_EQUAL_SIGN)
  return LexValue
}
{% endhighlight %}

The final state function we will look at is **LexValue**, which processes a value for a key. It stops when it reaches a newline character, emits a *TOKEN_VALUE*, and returns the state function **LexBegin**, which takes us back to the beginning.

{% highlight go %}
/*
This lexer function emits a TOKEN_VALUE with the value to be assigned
to a key.
*/
func LexValue(lexer *Lexer) LexFn {
  for {
    if strings.HasPrefix(lexer.InputToEnd(), lexertoken.NEWLINE) {
      lexer.Emit(lexertoken.TOKEN_VALUE)
      return LexBegin
    }

    lexer.Inc()

    if lexer.IsEOF() {
      return lexer.Errorf(errors.LEXER_ERROR_UNEXPECTED_EOF)
    }
  }
}
{% endhighlight %}

## What's Next?
In [part 3]({% post_url 2015-06-01-writing-a-lexer-and-parser-in-go-part-3 %}) we will look at the final portion where we build the basic parser which uses the lexer to assemble a data structure representing the INI file.
