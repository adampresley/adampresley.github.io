---
layout: post
title: ADAM moves along
date: 2006-09-25 20:28:00
author: Adam Presley
status: Published
tags: development
slug: adam-moves-along
---

Just a quick update. The ADAM language (Advanced Data Acquisition and
Manipulation) is coming along nicely. I've made several updates to the
lexical anaylizer to better handle variable and argument tokens. I have
also started on the interpreter engine. It appears to be executing code
correctly, so now I must start building the actual language commands.  
  
I've pulled out support for full expression parsing, as I don't see that
as necessary for a data ETL language. The purpose is to connect to data
stores, retrieve and manipulate the data, and the move it somewhere.
Once I get a working beta, I'll post the engine, the test tool, and some
code samples up here for examination. Cheers!
