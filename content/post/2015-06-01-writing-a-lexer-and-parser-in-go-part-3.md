---
author: "Adam Presley"
comments: true
date: 2015-06-01T00:00:00Z
tags: ["development", "golang"]
title: "Writing a Lexer and Parser in Go - Part 3"
slug: "writing-a-lexer-and-parser-in-go-part-3"
---

In [part one]({% post_url 2015-04-12-writing-a-lexer-and-parser-in-go-part-1 %}) of this series I introduced the concepts around lexical analysis and parsing. We took a look at the basics behind what makes up an INI file, and started setting up structures and constants that will help us perform lexical analysis, or *lexing*, on an input text, which is an INI file in our case. In [part two]({% post_url 2015-05-12-writing-a-lexer-and-parser-in-go-part-2 %}) we zoomed in and worked on the part of the process of lexical analysis. This process converted input text into a series of tokens.

In part three we will implement our interpreter. The *interpreter* will read the token stream, which is a channel of token structures, and create a structure in memory that represents a parsed INI file. Once the parser creates that structure we'll print it to the console as formatted JSON, essentially converting an INI file to JSON!

<!-- excerpt -->

The Model
---------
The parser is the component that begins the lexing process and starts reading tokens from the token stream. As it gets tokens it will keep track of the state we are in. As it does this we want it to start putting together a representation of the INI file into a memory structure. So the first order of business is to take a look at what that memory structure will look like.

We are going to break the INI file down into three structures. The first is the most basic thing that an INI file is designed to do, which is assign values to keys. Then we will have a structure for putting those key/value pairs into sections. And finally another structure will represent an INI file, which is nothing more than a file name and an array of sections. Now, for those key/value pairs that do not have have a section they will simply go into an unnamed section.

### /model/ini/IniKeyValue.go
{{< highlight go >}}
package ini

type IniKeyValue struct {
	Key   string `json:"key"`
	Value string `json:"value"`
}
{{< / highlight >}}

### /model/ini/IniSection.go
{{< highlight go >}}
package ini

type IniSection struct {
	Name          string        `json:"name"`
	KeyValuePairs []IniKeyValue `json:"keyValuePairs"`
}
{{< / highlight >}}

### /model/ini/IniFile.go
{{< highlight go >}}
package ini

type IniFile struct {
	FileName string       `json:"fileName"`
	Sections []IniSection `json:"sections"`
}
{{< / highlight >}}

The Parser
----------
Now to the fun part! Let's start working on the parser. The first thing we need to do is setup a variable that will contain our output result. This will be an **IniFile** structure.

{{< highlight go >}}
output := ini.IniFile{
   FileName: fileName,
   Sections: make([]ini.IniSection, 0),
}
{{< / highlight >}}

Now we will need some variables to track tokens, their values, and the state of parsing we are in. An example of state would be tracking the current section we are in when getting key/value pairs. Then we need to fire up the lexer!

{{< highlight go >}}
var token lexertoken.Token
var tokenValue string

/* State variables */
section := ini.IniSection{}
key := ""

log.Println("Starting lexer and parser for file", fileName, "...")

l := lexer.BeginLexing(fileName, input)
{{< / highlight >}}

Ok, so now that we have something to track state, and we have fired up the lexer, let's start grabbing tokens. To do this we will grab the next token from the stream. Then, if the token type is not a value, or **TOKEN_VALUE**, let's trim spaces. No need for those.

{{< highlight go >}}
for {
   token = l.NextToken()

   if token.Type != lexertoken.TOKEN_VALUE {
      tokenValue = strings.TrimSpace(token.Value)
   } else {
      tokenValue = token.Value
   }
{{< / highlight >}}

So if you recall our lexer will report an **EOF** if we reach the end of file and are done lexing. If so, let's record any section and key/value pair data we have, then break out of our loop.

{{< highlight go >}}
   if isEOF(token) {
      output.Sections = append(output.Sections, section)
      break
   }
{{< / highlight >}}

The final part of the parser function is to do something useful with the token and token value once we have it. There are three token types we are going to look at. The first is a section. If we encounter a section token, and we have been tracking a section and key/value pairs, record it. Then we reset our tracking values to prepare for any additional sections and key/value pairs. If the token type is a key track the key. That way when we get to the value token we can set the key's value, then add it to the section we are tracking.

{{< highlight go >}}
   switch token.Type {
   case lexertoken.TOKEN_SECTION:
      /*
       * Reset tracking variables
       */
      if len(section.KeyValuePairs) > 0 {
         output.Sections = append(output.Sections, section)
      }

      key = ""

      section.Name = tokenValue
      section.KeyValuePairs = make([]ini.IniKeyValue, 0)

   case lexertoken.TOKEN_KEY:
      key = tokenValue

   case lexertoken.TOKEN_VALUE:
      section.KeyValuePairs = append(section.KeyValuePairs, ini.IniKeyValue{
         Key: key,
         Value: tokenValue,
      })
      key = ""
   }
{{< / highlight >}}

Testing It
----------
Let's put it to the test! Go on and grab the [code from Github](https://github.com/adampresley/sample-ini-parser), put it into the ```../src/github.com/adampresley/sample-ini-parser``` folder inside your Go code folder. Then run ```go run ./*.go``` and let's see what you get! You should see something like the following screenshot.

![Sample Output](/assets/adampresley/images/posts/sample-ini-parser-output.png)

The code that generated that looks like this.

{{< highlight go >}}
sampleInput := `
   key=abcdefg

   [User]
   userName=adampresley
   keyFile=~/path/to/keyfile

   [Servers]
   server1=localhost:8080
`

parsedINIFile := parser.Parse("sample.ini", sampleInput)
prettyJSON, err := json.MarshalIndent(parsedINIFile, "", "   ")

if err != nil {
   log.Println("Error marshalling JSON:", err.Error())
   return
}

log.Println(string(prettyJSON))
{{< / highlight >}}

Final Thoughts
--------------
This has been a challenging, yet fun series to write. Lexing and parsing is a complex topic, and one that I am not terribly good at, as there is *so* much to learn. Even a simple strategy such as this can do amazing things though! I hope you've enjoyed this series, and happy coding!
