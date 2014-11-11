---
layout: post
title: On ColdFusion 7 Arrays
date: 2007-11-11 22:25:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: on-coldfusion-7-arrays
---

At [Ben Nadel's Blog](http://www.bennadel.com), which I enjoy reading simply because he has a
similar passion to me (digging into how and why it works under the
hood), he talked about calling Java functions in ColdFusion that require
an actual value array instead of a vector, which is what ColdFusion's
arrays are based on. He has an example, and then goes to show how
ColdFusion 8 made that task much easier.  
  
Only one thing I saw about it that struck me as odd was that he was
manually copying the values from the ColdFusion "array" (vector) to the
Java array. Didn't think that would be necessary, so I looked it up. Not
much different than Mr. Nadel's code, but a few lines shorter and if I
had to guess, the Java method to copy the data to the new array is
likely more efficient than a manual copy.  

{% highlight cfm %}
<!-------------------------------------------------
    ColdFusion array, which is actually a Vector.
-------------------------------------------------->
<cfset arrValue = ArrayNew(1)>

<cfset arrValue[1] = "What's up hot stuff?" />
<cfset arrvalue[2] = "You come here a lot?" />
<cfset arrValue[3] = "You from out of town?" />
<cfset arrValue[4] = "Want a little of this?" />
<cfset arrValue[5] = "How 'bout a little of that?" />

<!-----------------------------------------------------------------
Need to get the class of a string, and create our java array.
------------------------------------------------------------------>
<cfset stringClass = createObject("java", "java.lang.String").getClass() />
<cfset arrJavaValue = createObject("java", "java.lang.reflect.Array").newInstance(stringClass, arrayLen(arrValue)) />

<cfset arrValue.copyInto(arrJavaValue) />

<cfdump var="#arrValue#" />

<cfoutput>

<p>
#arrValue.GetClass().GetName()#
#arrValue.GetClass().GetSuperClass().GetName()#
#arrValue.GetClass().GetSuperClass().GetSuperClass().GetName()#
#arrValue.GetClass().GetSuperClass().GetSuperClass().GetSuperClass().GetName()#
#arrValue.GetClass().GetSuperClass().GetSuperClass().GetSuperClass().GetSuperClass().GetName()#
</p>
<h4>
Java String Array Class Hierarchy
</h4>
<p>
#arrJavaValue.GetClass().GetName()#
</p>
</cfoutput>
{% endhighlight %}
