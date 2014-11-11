---
layout: post
title: Barbecue Barcode, ColdFusion, and Size
date: 2009-05-18 19:30:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: barbecue-barcode-coldfusion-and-size
---

Swithin writes:

> Can the width and/or height parameters be passed in the
> o.getCode128() call?
>
> I am successfully passing the \#url.data\# as
> o.getCode128(\#url.data\#)
> but would like to also decrease the overall size of the barcode, while
> keeping the data the same. So I will need to decrease the
> width of the narrowest bar, and the height of the
> overall image. It appears that these have default settings that are
> built-in, and which cannot be passed in.
>
> Is that correct?
>
> If that is correct, and they cannot be sent in, then do you have any
> recommendation of another barcode generator that might allow such size
> adjustments?

Thanks for the question Swithin. I took a look into the API provided by
Barbecue Barcode and there is a way to control the individual bar's
width and height. There are some things to be aware of. First off each
barcode type may have minimum height requirements, so you might not
always get the results you expect.

The next noteworthy item is that you don't really have control over the
actual image's overall width and height. As you have probably noticed
just changing the image tag's width and height will invalidate the
barcode. I did try messing with the actual JComponent's
*setMaximumSize* and *setPreferredSize* methods, but those were
completely overlooked.

So here is an updated component that allows you to specify the bar
width, height, barcode resolution (for non-screen output, such as a
printer), and output format (gif, png, jpeg).

{% highlight cfm %}
<cfcomponent name="Barcode" displayname="BBQ Barcode" output="false">

	<cffunction name="getBarcode" returntype="any" access="remote" output="true">
		<cfargument name="data" type="string" required="true" />
		<cfargument name="barWidth" type="numeric" required="false" default="0" />
		<cfargument name="barHeight" type="numeric" required="false" default="0" />
		<cfargument name="resolution" type="numeric" required="false" default="0" />
		<cfargument name="outputFormat" type="string" required="false" default="jpeg" />

		<cfset var barcode = __getBarcode(argumentCollection: arguments) />

		<cfsetting showdebugoutput="false" />
		<cfset getPageContext().getOut().clearBuffer() /><cfcontent type="image/#arguments.outputFormat#" variable="#__getBarcodeImage(barcode, arguments.outputFormat)#" /><cfreturn />
	</cffunction>

	<cffunction name="__getBarcode" returntype="any" access="private" output="false">
		<cfargument name="data" type="string" required="true" />
		<cfargument name="barWidth" type="numeric" required="false" default="0" />
		<cfargument name="barHeight" type="numeric" required="false" default="0" />
		<cfargument name="resolution" type="numeric" required="false" default="0" />

		<cfset var barcode = createObject("java", "net.sourceforge.barbecue.BarcodeFactory").createCode128(arguments.data) />

		<!---
			If we have barWidth and barHeight data, pass that on. 0 indicates
			that we don't want to change the width/height.
		--->
		<cfif arguments.barWidth GT 0>
			<cfset barcode.setBarWidth(javaCast("int", arguments.barWidth)) />
		</cfif>

		<cfif arguments.barHeight GT 0>
			<cfset barcode.setBarHeight(javaCast("int", arguments.barHeight)) />
		</cfif>

		<!---
			Set the resolution. 0 indicates that we don't wish to change
			the default, which is usually 72 dpi. One may wish to change
			this for non-screen devices, such as a printer.
		--->
		<cfif arguments.resolution GT 0>
			<cfset barcode.setResolution(javaCast("int", arguments.resolution)) />
		</cfif>

		<cfreturn barcode />
	</cffunction>

	<cffunction name="__getBarcodeImage" returntype="any" access="private" output="false">
		<cfargument name="barcode" type="any" required="true" />
		<cfargument name="outputFormat" type="string" required="true" />

		<cfset var outStream = createObject("java", "java.io.ByteArrayOutputStream").init() />
		<cfset var bufferedImage = "" />

		<!---
			Write the barcode out to the output stream as a series of encoded bytes.
			The format is based on outputForamt. jpeg, gif, png.
		--->
		<cfswitch expression="#arguments.outputFormat#">
			<cfcase value="jpeg">
				<cfset bufferedImage = createObject("java", "net.sourceforge.barbecue.BarcodeImageHandler").writeJPEG(barcode, outStream) />
			</cfcase>

			<cfcase value="gif">
				<cfset bufferedImage = createObject("java", "net.sourceforge.barbecue.BarcodeImageHandler").writeGIF(barcode, outStream) />
			</cfcase>

			<cfcase value="png">
				<cfset bufferedImage = createObject("java", "net.sourceforge.barbecue.BarcodeImageHandler").writePNG(barcode, outStream) />
			</cfcase>
		</cfswitch>

		<cfreturn outStream.toByteArray() />
	</cffunction>

</cfcomponent>
{% endhighlight %}

And usage similar to before:

{% highlight cfm %}
<cfoutput>

<p>
	Displaying a barcode with the text "My First Barcode"
</p>

<p>
	<img src="barcode.cfc?method=getBarcode&outputFormat=gif&data=My First Barcode&barWidth=1&barHeight=50&resolution=72" alt="My First Barcode" />
</p>

</cfoutput>
{% endhighlight %}
