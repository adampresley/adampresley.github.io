---
layout: post
title: Generating XML in Groovy and Grails
date: 2010-11-22 23:26:00
author: Adam Presley
status: Published
tags: development groovy xml grails
slug: generating-xml-in-groovy-and-grails
---
I was doing a little work in a project that spans a rather broad
spectrum of the technology scale. One application in Groovy, the other
is a Grails app, while another piece is ColdFusion. Overall a very
interesting project.   
  
One of the things I was working on is a sort of RESTful API, which is
the portion written in Grails, and I found myself needing to craft an
XML response that is essentially a domain-class transformed. I found
this exercise in Grails to be quite simple.   
  
Groovy provides a class called **groovy.xml.MarkupBuilder**. This
class provides a DSL, or Domain Specific Language, for creating XML. The
syntax is actually quite easy and takes very little practice to become
natural. For the sake of demonstration let's pretend I have a
**Person** domain class, and I get a list of all the records from
the database. With this list of persons I want to build an XML file that
looks like this.  

{% highlight xml %}
<people>  
   <person id='1'>  
      <firstname>Imogene</firstName>  
      <lastname>Coffey</lastName>  
      <email>nec.tempus.scelerisque@montesnascetur.ca</email>  
   </person>  
   <person id='2'>  
      <firstname>Marsden</firstName>  
      <lastname>Mcbride</lastName>  
      <email>malesuada.malesuada.Integer@vitae.ca</email>  
   </person>  
   <person id='3'>  
      <firstname>Jescie</firstName>  
      <lastname>Vang</lastName>  
      <email>neque.venenatis@ipsum.edu</email>  
   </person>  
   <person id='4'>  
      <firstname>August</firstName>  
      <lastname>Hewitt</lastName>  
      <email>Nullam@MorbimetusVivamus.org</email>  
   </person>  
   <person id='5'>  
      <firstname>Raven</firstName>  
      <lastname>Gilmore</lastName>  
      <email>Sed@mollisInteger.org</email>  
   </person>  
</people>  
{% endhighlight %}

A pretty simple XML structure, but what does the domain class look
like?  

{% highlight groovy %}
package com.adampresley.groovy

class Person {
   String firstName
   String lastName
   String email = ""

   static constraints = {
      firstName(blank: false)
      lastName(blank: false)
   }

   @Override
   public String toString() {
      "${firstName} ${lastName}"
   }
}
{% endhighlight %}

So with that class, and an understanding of how I wanted the XML to
look, here's how to use the **MarkupBuilder** class to achieve
this.  
  
{% highlight groovy %}
package com.adampresley.groovy

import groovy.xml.MarkupBuilder

class TransformationsController {
   def xml = {
      def persons = Person.list()
      def writer = new StringWriter()
      def builder = new MarkupBuilder(writer)

      builder.people {
         persons.each { item ->
            builder.person(id: item.id) {
               firstName(item.firstName)
               lastName(item.lastName)
               email(item.email)
            }
         }
      }

      [ personXml: writer.toString() ]
   }
}
{% endhighlight %}

Cheers and happy coding!
