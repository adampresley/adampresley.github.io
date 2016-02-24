---
author: "Adam Presley"
date: 2010-10-08T02:44:00Z
tags: ["development", "groovy", "mongodb", "grails"]
title: "My Grails and MongoDB Experience (so far)"
slug: "my-grails-and-mongodb-experience-so-far"
---

Slowly but surely I have been spending time learning how to develop web
applications in Grails, a web application framework built on top of
Groovy, Spring, and Hibernate. Last night I spent a good deal of time
and effort trying to fit MongoDB into this framework stack, and met with
quite a bit of resistance. I found that I didn't quite like the existing
Grails plugins for MongoDB, so I set off to do this in a little more
"manual" way.

Once I had my Grails project setup I first set out to download the [Java
driver](http://github.com/mongodb/mongo-java-driver/downloads) for MongoDB. The nice folks who make this lovely
document-based database product provide this already, so that was a
no-brainer. There are plenty of tutorials on how to use this driver so I
won't delve into that.

After getting my classpath adjusted accordingly I set out on how to make
and persist my Grails domain classes. The first annoyance is that I
could not put my classes in the actual *domain* folder, but instead
had to place them into *src/groovy*. This appears to be a Grails
limitation because everything in the *domain* folder expects to be
persisted by Hibernate, and since I am not using Hibernate that caused
an issue. Allegedly this is being addressed in the next release of
Grails. I tried a number of techniques to persist my classes to MongoDB
but found a lot of walls.

Enter [Morphia](http://code.google.com/p/morphia/). This nifty jewel is a type-safe library for mapping
Java objects to MongoDB. With Morphia I only have to add very little to
describe my class so that it can be persisted to MongoDB. Here's a
sample of a class the represents a user's Role.

{{< highlight groovy >}}
import com.google.code.morphia.annotations.*
import org.bson.types.*

@Entity
class Role {
   @Id ObjectId id
   String roleName
   boolean isSuperAdmin = false
   int sortOrder = 0
}
{{< / highlight >}}

You'll notice I added annotations for the class itself to describe that
it is a Mongo Entity, and another for the ID field. That's it. With this
in place it is a simple matter to persist the object.

{{< highlight groovy >}}
def mongo = new Mongo("127.0.0.1")
def morphia = new Morphia()
def ds = morphia.createDatastore(mongo, "myDB")

morphia.map(Role.class)

def newRole = new Role(roleName: "user", isSuperAdmin: false, sortOrder: 1)
ds.save(newRole)
{{< / highlight >}}

Easy enough to do and worked great. From here I created an additional
class for a User that has a relationship to the Role class. In MongoDB
you can either embed an object that has a relationship, or you can
reference. In this case I opted to reference.

{{< highlight groovy >}}
import com.google.code.morphia.annotations.*
import org.bson.types.*

@Entity
class User {
   @Id ObjectId id
   String userName = ""
   String email = ""
   String password = ""
   String firstName = ""
   String lastName = ""
   @com.google.code.morphia.annotations.Reference List<role> roles
}
{{< / highlight >}}

As you can see my user class has most of what you would expect from one.
The user may, however, have one or more roles in the system, so I've
created a List of Role objects, and annotated it with the @Reference
annotation. This tells Morphia that when persisting this object save it
with a reference to the listed roles.

After getting this all to jive I then decided I wanted to hide the
connection and data store details and make it simpler to consume. At
this point I created a **Database** class that handles all the
connection code, as well as mapping the objects to Morphia. To make it
easy to instantiate I used Grail's Spring DSL to allow Grails to manage
instantiation and dependencies. To do this I opened up the
*conf/spring/resources.groovy* file and edited it to look like so.

{{< highlight groovy >}}
// Place your Spring DSL code here
beans = {
   /*
    * Database and DAO beans
    */
   database(com.myproject.data.Database)
}
{{< / highlight >}}

Now inside of a controller or my bootstrap I can simply use **def
database** and I will get my database object that handles connecting
and mapping for me (and has a handy method to get the datastore back
too). Once this was in place I wanted to utilize the DAO class
extensions offered by Morphia. To do this I created a class that looks
like so.

{{< highlight groovy >}}
import com.myproject.data.Database
import com.mongodb.Mongo
import com.google.code.morphia.*
import org.bson.types.*

class RoleDAO extends DAO<role, objectid=""> {
   public RoleDAO(Database database) {
      super(database.ds)
   }
}
{{< / highlight >}}

After creating this I then wired it into Spring similar to before. The
biggest difference is that I need to pass the database object to this
DAO in the constructor. Here's how that works.

{{< highlight groovy >}}
// Place your Spring DSL code here
beans = {
   /*
    * Database and DAO beans
    */
   database(com.myproject.data.Database)
   roleDAO(com.myproject.Role, database)
   userDAO(com.myproject.User, database)
}
{{< / highlight >}}

With the DAO and database connectivity all wired up to Spring, I could
now create some sample data in my bootstrap.

{{< highlight groovy >}}
def database
def roleDAO, userDAO

def init = { servletContext -&amp;gt;
   roleDAO.deleteByQuery(roleDAO.createQuery())
   userDAO.deleteByQuery(userDAO.createQuery())

   /*
    * Create all roles
    */
   def superAdminRole = new Role(roleName: "superadmin", sortOrder: 1, isSuperAdmin: true)
   def adminRole = new Role(roleName: "admin", sortOrder: 2, isSuperAdmin: false)
   def userRole = new Role(roleName: "user", sortOrder: 3, isSuperAdmin: false)

   roleDAO.save(superAdminRole)
   roleDAO.save(adminRole)
   roleDAO.save(userRole)

   /*
    * Make a super user
    */
   def user = new User(
      userName: "admin",
      email: "test@test.com",
      password: "password",
      firstName: "John",
      lastName: "Smith",
      roles: [ superAdminRole ]
   )

   userDAO.save(user)
}
{{< / highlight >}}

So far this experience has been interesting. And by interesting I mean a
tad frustrating, but very educational. I'll post more as I learn more
about this experience. Happy coding!
