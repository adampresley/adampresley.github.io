---
layout: post
title: Coldfusion Soundex Algorithm Component
date: 2008-07-18 11:43:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: coldfusion-soundex-algorithm-component
---
## Notice: Code is no longer here

Although this has been done a million times I took a little time
(wasting time mostly) to cook up a simple ColdFusion component that
implements a basic soundex algorithm (see
<http://www.creativyst.com/Doc/Articles/SoundEx1/SoundEx1.htm> for more
info on the reference I used). The component provides a simple interface
to add "dictionary" items, and compare subject text against the
dictionary items within a specific threshold for potential soundex
matches. Without going into detail about the component implementation, here's an
example of how it is used. Let's say that you take an input, and want to
show words that may be like what was entered.  
  
{% highlight cfm %}
<!---
	Instantiate the soundex component, and a structure for
	storage.
--->
<h1>ColdFusion Soundex Function</h1>

The word: <strong>#s.word#</strong>
The threshold: <strong>#s.threshold#</strong>

Did you mean? 
<ul>
	<li><strong>#s.result[index].candidate#</strong></li>
</ul>
{% endhighlight %}

In this example we instantiate the soundex component and add a series of
words to the dictionary. Behind the scenes the soundex for each word is
calculated and stored. We then define a structure to hold our word,
threshold for matches, and tell the component to return potential
matches using the **returnSoundexMatches** function.  
  
Anyway, just a fun little trip back into one of the older form of index
techniques. The component can be found here.
<http://www.adampresley.com/downloads/code/coldfusion/soundex.zip>
