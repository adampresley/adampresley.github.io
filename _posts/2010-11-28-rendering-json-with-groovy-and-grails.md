---
layout: post
title: Rendering JSON with Groovy and Grails
date: 2010-11-28 23:25:00
author: Adam Presley
status: Published
tags: development groovy json grails
slug: rendering-json-with-groovy-and-grails
---

The other day I blogged about generating XML from domain classes in
Grails. Today I want to show how to do the same thing but with JSON
instead. JSON is widely used and accepted as a great way to exchange
information in AJAX and RESTful applications, and Grails makes it
mind-numbingly simple to do this.

Grails provides a class called \*\*grails.converters.JSON\*\*. This
class provides a DSL, or Domain Specific Language, for creating JSON, as
well as automatic marshaling of objects to JSON. In this sample I will
demonstrate the automatic marshaling technique, and show how to return
the JSON string back to the view, and even how to render it to the HTTP
output for AJAX applications. For the sake of demonstration let's
pretend I have a \*\*Person\*\* domain class, and I get a list of all
the records from the database. With this list of persons I want to build
an XML file that looks like this.

	:::javascript
	[
		{"class":"com.adampresley.groovy.Person","id":1,"email":"nec.tempus.scelerisque@montesnascetur.ca","firstName":"Imogene","lastName":"Coffey"},
		{"class":"com.adampresley.groovy.Person","id":2,"email":"malesuada.malesuada.Integer@vitae.ca","firstName":"Marsden","lastName":"Mcbride"},
		{"class":"com.adampresley.groovy.Person","id":3,"email":"neque.venenatis@ipsum.edu","firstName":"Jescie","lastName":"Vang"},
		{"class":"com.adampresley.groovy.Person","id":4,"email":"Nullam@MorbimetusVivamus.org","firstName":"August","lastName":"Hewitt"},
		{"class":"com.adampresley.groovy.Person","id":5,"email":"Sed@mollisInteger.org","firstName":"Raven","lastName":"Gilmore"}
	]

A pretty simple JSON structure. Now let's look at that Person domain
class. You may remember this from my Grails and XML post.

	:::groovy
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

Nothing has changed here. Now let's see how we can use the power of
Grails to easily transform an array of Person objects into JSON data.

	:::groovy
	package com.adampresley.groovy

	import grails.converters.JSON

	class TransformationsController {
		def json = {
			def persons = Person.list()
			println (persons as JSON)
			[ personJSON: (persons as JSON) ]
		}

		def renderJson = {
			def persons = Person.list()
			render (persons as JSON)
		}
	}

The above controller has two methods, one for returning a JSON string,
and the other renders JSON to the HTTP output. As you can see it is
quite possibly the simplest thing you've ever done in your life. Cheers and
happy coding!
