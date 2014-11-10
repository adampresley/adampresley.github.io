---
layout: post
title: Finding Specific Text in a String using ColdFusion and Java
date: 2009-07-15 17:24:00
author: Adam Presley
status: Published
tags: development coldfusion regex
slug: finding-specific-text-in-a-string-using-coldfusion-and-java
---

A question was posted on the Dallas/Ft. Worth ColdFusion User Group
today.  

> I'm trying to parse a text file to find a particular string, then
> extract that string and approximately 100 characters past that.
  
So I've cooked up a little code sample that uses the Java regular
expressions classes. The ColdFusion regular expression methods are quite
nice, and will serve most purposes, but for tasks like this where I have
to run a regex pattern repeatedly speed is important. Especially if you
are working over a large file of data, for example.  
  
So what I've done is use the **java.util.regex.Pattern** class and the
**java.util.regex.Matcher** class to do the job. The **Pattern**
class allows you to compile a regular expression string you intend to
use over and over, as well as setup some options on how the regex engine
is to behave.  
  
The **Matcher** class allows you to iteratively match the pattern
against a string, allowing you to find multiple matches, one at a time.
So what you do is ask the matcher if you have any more string to search,
and if you do, tell it to find the next match. If it finds a new match
you can retrieve information about what the start and end positions of
the match are, and more. So let's see that in action.  
  
    :::cfm
    <cfset haystack = "Hello. My name is Adam. It is nice to meet you. How are you? This is an example of a regex "&
    "that finds something specific as a string to start, then followed by 100 or so characters. How many "&
    "will we have? I am a giant noob who is writing lengthy text to try and see if we've hit our "&
    "something specific threshold. Now we should have two matches." />

    <!---
        Create the pattern. The compiled pattern
        needs to be case insensitive, and dot matches
        all.

        See: http://72.5.124.55/javase/6/docs/api/java/util/regex/Pattern.html
    --->
    <cfset patternObj = createObject("java", "java.util.regex.Pattern") />
    <cfset matchStart = "something\sspecific" />

    <cfset pattern = patternObj.compile(
        "(#matchStart#.{1,100})",
        bitor(patternObj.CASE_INSENSITIVE, patternObj.DOTALL)
    ) />

    <!---
        Get a matcher object to match our pattern against 
        our input.
    --->
    <cfset matcher = pattern.matcher(haystack) />

    <!---
        Continue matching till we have no more matches.
        When we find a match get the start, end, and value
        of the match.
    --->
    <cfset local = [] />
    <cfset index = 1 />

    <cfloop condition="!matcher.hitEnd()">
        <cfif matcher.find()>
            <cfset local[index] = {
                start = matcher.start(),
                end = matcher.end(),
                value = haystack.subsequence(matcher.start(), matcher.end())
            } />

            <cfset index++ />
        </cfif>
    </cfloop>

    <cfdump var="#local#" />
  
As you can see at the beginning I am simply setting up a large string
with our data to search against. From here we create a pattern object,
followed by what our starting match criteria is. So the variable
**matchStart** is what string we wish to start the match with. From
here we use the *compile* method to compile the regular expression. In
this method the second argument is taking two integers,
CASE_INSENSITIVE and DOTALL and ORing the together. This tells the
pattern that we don't care about case, and the DOT character in the
regex matches all lines.  
  
After this we ask the pattern for a matcher object, telling it also that
we wish to work against the haystack string. What we then do with it is
to loop until the matcher tells us we've hit the end of our search
string, as seen on line 35.  
  
The *find* method asks the matcher to search the haystack for our regex
match. If we have a successful match (because *find* returns boolean),
we want to get the start and end positions, as well as the matched
value, and stuff them into a structure, and place that structure into an
array.  
  
From here you will have an array of matches, their start and end
offsets, that you may then use to your heart's content. Happy coding!
