---
layout: post
title: ADAM Gets an Overhaul
date: 2006-12-04 15:58:00
author: Adam Presley
status: Published
tags: development
slug: adam-gets-an-overhaul
---

Ok, a few out there know that I've actually been working on my ADAM
language (Advanced Data Acquisition and Manipulation) for some time. I
had actually had a lexical analyzer, byte code compiler, and base
interpreter already built. The only thing I didn't like was the
definition of the language. It was leaning toward an imperative
language, and this is not exactly the vision I had for it.  
  
So ADAM is going for a complete reconstruction. Not only is the language
definition closer to what I was envisioning, but I am now moving toward
a .NET compiled language. This will be accomplished by changing the byte
code compiler to compile to MSIL instead. MSIL, for those who do not
know, is the Microsoft Intermediate Language. This is what all .NET
language compile to before being compiled to an executable. MSIL
resembles C++ in that the syntax is very similar, complete with classes,
and the order of defining methods and variables. Other parts are similar
to assembly language in that you must push all your function parameters
prior to calling, or JUMPING in assembly, your function call.  
  
Ok, so I've got a better example of the language structure, as well as a
sample breakdown of the lexical analysis output tree.  

```text  
create source Employees  
begin  
"excel" -\> type  
"employees1099.xls" -\> filename  
"Sheet1" -\> sheet  
"FirstName != ''" -\> criteria  
end  
```

```text  
COMMAND : CREATE SOURCE  
PARAMETER : Employees  
IDENTIFIER : BEGIN  
STRING : "excel"  
DELIMITER : -\> (assignment)  
IDENTIFIER : type  
STRING : "employees1099.xls"  
DELIMITER : -\> (assignment)  
IDENTIFIER : filename  
STRING : "Sheet1"  
DELIMITER : -\> (assignment)  
IDENTIFIER : sheet  
STRING : "FirstName != ''"  
DELIMITER : -\> (assignment)  
IDENTIFIER : criteria  
IDENTIFIER : END  
```
 
This would open up an excel sheet as a data source. The objective here
is to make the language as "descriptive" as possible. Stay tuned for
more work on this.  
  
I now have to spend a bit more time defining the token representation of
the language, the different elements it will break down into, and then
the structures that will house this data. Then I must map out how to
move this to MSIL efficiently.
