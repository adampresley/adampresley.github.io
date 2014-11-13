---
layout: post
title: Webservice WSDL Document Consumption in ColdFusion
date: 2009-12-30 08:10:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: webservice-wsdl-document-consumption-in-coldfusion
---
This question was posted on the Adobe ColdFusion forums (found here at
<http://forums.adobe.com/thread/546708?tstart=0>).

> Does anyone know of a service for this compatible with CF? I need to
> display an amount for Korean Won and USD on a checkout form and the
> service at
> http://www.xmethods.net/sd/2001/CurrencyExchangeService.wsdl doesn't
> exist anymore. I searched Google for a while but didn't find much.
>
> Thanks!

The issue was in understanding how to read a raw WSDL (Web Service
Description Language) document. I'll be the first to admit that these
can be a bit daunting, but after you know a little about how they are
typically structured, you can surmise what methods and their arguments
are available to you.

The WSDL in question can be found here at
<http://www.webservicex.net/CurrencyConvertor.asmx?WSDL>. Go ahead and
open that and take a look. The first thing we can see about this
document is at the top with the tag declaration. Inside here we see an
element declaration (as seen by the tag) named ConversionRate. This is a
clue into what methods may be available.

To be certain you want to find definitions. These are the actual methods
that are defined in the service. Looking at this document we can see
three distinct methods: CurrencyConvertorSoap, CurrencyConvertorHttpGet,
and CurrencyConvertorHttpPost. All of these methods have the same name,
"ConversionRate". This is seen by looking at the tag under the tag.

The reason for three methods with the same operation name becomes more
apparent when we go down and examine the tags. These tags actually
specify the details of HOW a given method can be called. You'll notice
that there are three bindings (actually there are four, but one is for a
different SOAP binding) here. One for SOAP, POST, and GET. Each of those
points to the appropriate portTypes that we saw above. How can we tell?
Take a look at the "type" attribute on the tag. It will point to the
portType definitions above.

Ok, that's cool. So we know what methods are available, but how do we
know what arguments to pass to these methods? Let's take a look at the
SOAP method version of "ConversionRate". At the bottom of that
definition you will see and tags. These specify what inputs and outputs
are expected. Input is what we wish to look at first.

In this method the tag contains a message attribute with the value of
"tns:ConversionRateSoapIn". This tells the service that the input, or
arguments, expected are defined in the "ConversionRateSoapIn" element.
Scroll up just a tad and you will see it's definition in the tag that
has a name attribute of "ConversionRateSoapIn". To get more confusing,
this contains a tag named with an "element" attribute of
"tns:ConversionRate". Remember that element tag we talked about at
first? That is what this attribute is referring to. Go ahead and take a
look at that.

The "ConversionRate" element is a complex type, which is a sequence of
arguments. There are two actually: FromCurrency, and ToCurrency. Sweet,
there are our arguments. Notice also that their type is "tns:Currency".
That is actually just below this element and looks like "". This is
actually a list of valid values for our arguments FromCurrency and
ToCurrency.

Great, so that's a lot of work. Welcome to the world of Web Services!
Now let's look at a small snippet of ColdFusion code that actually uses
this service!

{% highlight cfm %}
<cfset service = createObject("webservice", "http://www.webservicex.net/CurrencyConvertor.asmx?WSDL") />

<cfset fromCurrency = "AUD" />
<cfset toCurrency = "USD" />

<cfset result = service.ConversionRate(fromCurrency, toCurrency) />
<cfdump var="#result#" />
{% endhighlight %}

Woah, that was easy! Just create the object as a type of "webservice",
give it the URL to the WSDL, and call the method "ConversionRate" and
pass in two strings. Here we are getting the conversion rate from
Australia currency to American USD.

I hope this sheds a LITTLE light on the loveliness that is WSDL
documents. Happy coding!
