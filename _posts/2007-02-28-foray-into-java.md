---
layout: post
title: Foray into Java
date: 2007-02-28 04:00:00
author: Adam Presley
status: Published
tags: development sql java
slug: foray-into-java
---

So recently I've decided to delve into the world of Java development,
or, more specifically, desktop Java development. So far I am liking it.
My first task was to write a small application that would accept two
dates as input and spit out a tab-separated report of first and last
journal entries in a system, broken down by each day. Turned out to be a
good learning experience, and useful to my bosses. :)  
  
So if you've ever wanted to know how to connect to and use a Microsoft
SQL Server 2000 box in Java, perhaps you will find this useful. First
thing to do? Download the Microsoft JDBC Type 4 driver [here](http://www.microsoft.com/downloads/details.aspx?FamilyID=86212d54-8488-481d-b46b-af29bb18e1e5&DisplayLang=en). You
don't need the setup program, just the JAR files. So extract those to a
location you can remember.  

Now for my development I am using NetBeans 5.5. I highly recommend this
product. Not only does it **ROCK**, but it is **FREE**! To get SQL 2000
working in your Java app you will need to add the SQL 2000 JAR files
into your project classpath settings in NetBeans. Once you have done
this, here is how you can connect to and read records from a table in
Microsoft SQL Server 2000.  

{% highlight java %}
DriverManager.registerDriver(new SQLServerDriver());
Connection connection = DriverManager.getConnection("jdbc:microsoft:sqlserver://localhost:1433;databaseName=MyDb", "userName", "password");

PreparedStatement qry = connection.prepareStatement("SELECT * FROM myTable");
ResultSet table = qry.executeQuery();

while (table.next()) {
   someValue1 = table.getString("someColumn1");
   someValue2 = table.getDate("someColumn2");
}

table.close();
qry.close();
connection.close();
{% endhighlight %}

Obviously you will need to import the correct packages and such, but you
get the idea.
