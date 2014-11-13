---
layout: post
title: Barbecue revisited.
date: 2007-10-12 11:00:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: barbecue-revisited
---
I made a post recently about using Barbecue barcode software in
ColdFusion. In my example I instantiated the CFC object and called the
*getCode128* function as the source of an image tag. With a bit of
experimentation you will find that although this produces an image...
that's **ALL** it produces. Why? Cause we essentially hijack the content
type of ColdFusion's output buffer and set it to *image/jpeg*. It
removes all other content from the buffer, and **only** sends over the
image. The fix?  
  
## The Component
{% highlight cfm %}
<cfcomponent>

   <cffunction name="getCode128" returntype="any" access="remote" output="true">
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

  
## Usage
{% highlight cfm %}
<cfoutput>

<img src="barcode.cfc?method=getCode128&data=My First Barcode" />

</cfoutput>
{% endhighlight %}

So, as you can see in this case I am calling the CFC method remotely in
the image source attribute forcing a separate HTTP request. This also
required me to make the access to the getCode128 function to remote.
Hope it helps! Cheers!

