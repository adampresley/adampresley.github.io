---
layout: post
title: My Resume - The Code and Summary
date: 2010-09-10 10:24:00
author: Adam Presley
status: Published
tags: development coldfusion couchdb
slug: my-resume-the-code-and-summary
---

Back by popular demand I have put the code for my online resume
application up for [download](http://dl.dropbox.com/u/5726689/blog-downloads/resume.adampresley.com.zip)!
Not too long ago I re-created my online resume in Railo using the wonderful small FW/1 framework. For the
persistence layer I used Apache CouchDB and my ColdFusion wrapper for
the Java CouchDB4J library, CouchDB4CF. Since then I have received
numerous requests to put the code up online. So this post is doing just
that. If you want to look over the code, the first order of business is
to [download](http://dl.dropbox.com/u/5726689/blog-downloads/resume.adampresley.com.zip) it.

To begin extract the contents of the ZIP file into any directory of your
choice. There should be two sub-directories, **application** and
**tomcat**. You shouldn't need to do much of anything in the
**application** directory, but you will need to do a slight
configuration change in the **tomcat/conf** directory. In this
directory open up **server.xml** and near the bottom find the XML
node for the localhost host. There is a node inside of this named
**Context**, and the **docBase** path will need to be modified
to match where you have extracted the ZIP file. Save those changes.

Next you will need to actually have an instance of the Apache CouchDB
server running. It is beyond the scope of this blog post to detail how
to actually setup Couch, but there are enough tutorials out there to get
you started. In fact you will likely start at
<http://couchdb.apache.org/>. Get Couch installed and setup. Once you've
done this your will need to log into Futon and create a database called
"resume". From here you probably want some data, so I have included a
copy of my resume JSON object that can be inserted using the command
line (CURL is a nice option). Here is a [blog post](http://morethanseven.net/2007/12/11/using-curl-to-play-with-couchdb.html)
on using CURL to put data into Couch. You can find the JSON data in the
**application/doc** directory. There are two documents to import.

If you thought there was more than that then I am sorry to say that you
should just be able to start up the app. First make sure Couch is
running. Then browse to the **tomcat/bin** and run either
**startup.bat** or **startup.sh**, depending on your OS. From
here you should be able to browse to **http://localhost:8080** and
see my resume. To get to the Railo web administrator you can just browse
to **http://localhost:8080/railo-context/admin.cfm** and use the
password "password".

That's the summarized breakdown. If anyone has any questions please do
not hesitate to contact me and I will try to answer as swiftly as my
work schedule allows! Happy coding!
