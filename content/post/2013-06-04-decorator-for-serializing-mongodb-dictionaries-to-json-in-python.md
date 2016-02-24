---
author: "Adam Presley"
date: 2013-06-04T19:48:00Z
tags: ["python", "mongodb"]
title: "Decorator for Serializing MongoDB Dictionaries to JSON in Python"
slug: "decorator-for-serializing-mongodb-dictionaries-to-json-in-python"
---

Before you read this I'm going on the record stating that this problem
has likely been solved hundreds of times before. So here is yet another
take on how I am managing to serialize Python objects that are returned
as a result of a query to a [MongoDB](http://www.mongodb.org/) database. The biggest issue I
was running into is when you try to serialize something like a
dictionary that has a MongoDB ID in it, the data type of that ID is
*bson.objectid.ObjectId*, and apparently that data type does not
serialize to JSON well without running something like **str(_id)**.

<!-- excerpt -->

To address this I first started poking around the inter-tubes and found
a really nice answer to a post written by Shabbyrobe that will
recursively grab the keys and values of a class object so you can
convert it to a dictionary
(<http://stackoverflow.com/questions/1036409/recursively-convert-python-object-graph-to-dictionary>.
This was a good start and I only needed a couple of small modifications
to ensure that I check for any value that is of type *ObjectId* got
converted correctly to a string.

The next thing I wanted was to make sure that I didn't have to do
anything silly like this.

{{< highlight python >}}
return [myService.serialize(item) for item in myService.getMongoRecords()]
{{< / highlight >}}

To address this I put this nifty recursive object walker method into a
decorator. With a decorator I could decorate any route method in my
[Bottlypy](http://bottlepy.org/docs/dev/) application with **@ajax** and it would run this recursive
object walk and make sure any response I return from a method decorated
would have MongoDB IDs properly converted to string. Here's what that
decorator code looks like.

{{< highlight python >}}
from bson.objectid import ObjectId

def ajax(fn):
   def wrapper(*args, **kwargs):
      def _toDict(obj):
         if isinstance(obj, dict):
            for key in obj.keys():
               obj[key] = _toDict(obj[key])

            return obj

         elif hasattr(obj, "__iter__"):
            return [_toDict(item) for item in obj]

         elif hasattr(obj, "__dict__"):
            return dict([(key, _toDict(value)) for key, value in obj.__dict__.iteritems() if not callable(value) and not key.startswith("_")])

         else:
            if isinstance(obj, ObjectId):
               return str(obj)
            else:
               return obj

      result = fn(*args, **kwargs)
      return _toDict(result)

   return wrapper
{{< / highlight >}}

To use this in my Bottlepy application I would have a method for an AJAX
response route that looks something like this.

{{< highlight python >}}
@route("/ajax/contact/:id", method="GET")
@ajax
def ajax_getContact(id):
   contactService = request.factory.getContactService()
   return contactService.read(id=id)

@route("/ajax/contacts", method="GET")
@ajax
def ajax_getContacts():
   contactService = request.factory.getContactService()
   return contactService.list()
{{< / highlight >}}

Notice how each of these methods are decorated with our cool decorator?
That ensures that the objects and lists I return are ready for proper
JSON serialization that is automatically done by the Bottlepy framework.
I hope this helps somebody. Not 100% sure this is a great way to do it,
but it is solving my immediate problem. Cheers, and happy coding!
