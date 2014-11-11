---
layout: post
title: Grails, MongoDB, and the Domain Class
date: 2010-09-19 23:50:00
author: Adam Presley
status: Published
tags: development groovy grails
slug: grails-mongodb-and-the-domain-class
---

While doing some experimentation with Groovy and Grails tied to a
MongoDB database I was running into difficulties. I first tried the two
plugins that are available for Grails and Mongo, but I just didn't find
them to work quite the way I liked, and didn't feel like they fit the
GORM paradigm.   
  
So I decided to old-school it and just use the Java MongoDB library and
persist objects the old fashioned way. I quickly ran into a problem
where the **Customer** class I created in the *domain* directory
could not be instantiated. I simply got a null value as the returned
object. Clearly trying to persist a null to Mongo was not going to work!
As a result I posted a message to the Dallas Groovy user group and
someone suggested I move my class to the */src/groovy* directory. This
worked like a charm! So here is my class.  

{% highlight groovy %}
class Customer {
   def name = ""
   def address = ""
   def city = ""
   def state = ""
   def zip = ""
}
{% endhighlight %}

And then here is my Bootstrap where I am simply setting up some sample
data using the MongoDB Java drivers and the JSON-lib library.  
  
{% highlight groovy %}
import mongodbtest1.*
import net.sf.json.groovy.GJson;
import com.mongodb.*
import net.sf.json.*

class BootStrap {
   def init = { servletContext ->
      servletContext["mongo"] = new Mongo("localhost", 27017)
      servletContext["test"] = servletContext["mongo"].getDB("test")
      servletContext["collections"] = [
         customer: servletContext["test"].getCollection("Customer")
      ]

      /*
       * Clear out all records
       */
      servletContext["collections"].customer.drop()

      /*
       * Create some test records yo.
       */
      servletContext["collections"].customer.insert(JSONObject.fromObject(new Customer(
         name: "Adam",
         address: "111 MyStreet Dr.",
         city: "Allen",
         state: "TX",
         zip: "77777"
      )) as BasicDBObject)
   }

   def destroy = {
   }
}
{% endhighlight %}
  
And with that I can log into the Mongo console and do
**db.Customer.find()** and see the document I just inserted. Woot!
