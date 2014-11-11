---
layout: post
title: Using Barbecue Barcode in ColdFusion
date: 2007-10-03 10:54:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: using-barbecue-barcode-in-coldfusion
---

I was approached today by a coworker about integrating the open source
Java-based barcode library, [Barbecue](http://barbecue.sourceforge.net/),
into ColdFusion. He wanted to use this library to prototype a project
concept for adding barcode support to certain parts of our application.
A little bit of play and here's how it works.

The first thing you will need to do is extract the JAR file from the
download ZIP file (the file I used was barbecue-1.5-beta1.jar). Put this
file into your {ColdFusionRoot}\runtime\lib (for single server
installs). You will need to restart the ColdFusion service for CF to
pick that up.

Now lets do some code. First lets create a component called
*barcode.cfc*. And in there we will have a function called
**getCode128()**, which will return a Code 128 barcode that dynamically
switches between character sets to give the smallest possible encoding.
It will look like so.

{% highlight coldfusion %}
<cfcomponent>
	<cffunction name="getCode128" returntype="any" access="public" output="true">
		<cfargument name="data" type="string" required="true" />

		<cfsetting showdebugoutput="false">

		<cfscript>
			outStream = createObject("java", "java.io.ByteArrayOutputStream").init();

			//-----------------------------------------------------------
			// Create the barcode, and draw it to a buffered image class.
			//-----------------------------------------------------------
			barcode = createObject("java", "net.sourceforge.barbecue.BarcodeFactory").createCode128("#arguments.data#");
			bufferedImage = createObject("java", "net.sourceforge.barbecue.BarcodeImageHandler").writeJPEG(barcode, outStream);
		</cfscript>

		<!--- Return the information as streaming bytes of type image/jpeg --->
		<cfset getPageContext().getOut().clearBuffer()><cfcontent type="image/jpeg" variable="#outStream.toByteArray()#"><cfreturn>
	</cffunction>
</cfcomponent>
{% endhighlight %}

The code is pretty well documented so it should be pretty clear what we
are doing here. Now, to use it we will simply create an instance of our
new component, and call the **getCode128()** function as the source to an
image tag in HTML. Like so.

{% highlight coldfusion %}
<cfoutput>

<cfset o = createObject("component", "barcode")>
<img src="#o.getCode128('My First Barcode')#" />

</cfoutput>
{% endhighlight %}

So there you have it. Give it a go! It was fun!
