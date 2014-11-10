---
layout: post
title: Parsing PHP-style Placeholders for SQL Scripts
date: 2008-09-04 19:12:00
author: Adam Presley
status: Published
tags: development php
slug: parsing-php-style-placeholders-for-sql-scripts
---

If you've ever used PHP, you've probably used, or at least seen, the
[PRINTF](http://us2.php.net/manual/en/function.printf.php) and/or 
[SPRINTF](http://us2.php.net/manual/en/function.sprintf.php) functions. These oldie, but goodie,
functions are stolen... err... borrowed from the old [C language](http://en.wikipedia.org/wiki/C_language)
world. Both functions support the abililty to have placeholders, which
when evaluated, are replaced with the value in a comma-delimited list in
the appropriate position. For example:  

    :::php
    printf("My name is %s. I am %i years old", $name, $age);
  
This code, given that **$name** equals "Adam", and **$age** equals
"30" will output "My name is Adam. I am 30 years old". A nifty feature.  
  
Below is a bit of code that provides a class that will take a given
input SQL string, an array of arguments, and spit out a parsed SQL
string. An example of usage would look like so:  

    :::php
    $userId = 30;$parser = new SqlScriptParser("SELECT name, email, active FROM users WHERE userId=%i", array($userId));$result = $parser->Parse();

The resulting string would look like:  
  
**SELECT name, email, active FROM users WHERE userId=30**  
  
The code:
http://www.adampresley.com/downloads/code/php/sqlScriptParser.zip

Note: This ZIP file got lost over time.