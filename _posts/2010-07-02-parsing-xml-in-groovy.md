---
layout: post
title: Parsing XML in Groovy
date: 2010-07-02 23:44:00
author: Adam Presley
status: Published
tags: development groovy
slug: parsing-xml-in-groovy
---
I've been doing a little more work with Groovy lately, and it still
continues to impress me on how expressive and easy some things are to do
in this language. Last night I was working with a web service that
returns XML and I needed to parse this XML response and do some work
with it. Talk about easy! So now I'm going to show you a small example
of parsing an XML document of movies, and we'll display some movie
information in the console. The purpose: to see how simple it is to
manipulate and read XML documents in Groovy!  

{% highlight xml %}
<movies>
    <movie name="The Matrix">
        <year>1999</year>
        <directors>
        <director name="Andy Wachowski" />
        <director name="Lana Wachowski" />
        </directors>
        <plotsummary><![CDATA[A computer hacker learns from mysterious rebels about the true nature of his reality and his role in the war against the controllers of it.]]></plotSummary>
    </movie>
    <movie name="Kill Bill: Vol. 1">
        <year>2003</year>
        <directors>
        <director name="Quentin Tarantino" />
        </directors>
        <plotsummary><![CDATA[The Bride wakes up after a long coma. The baby that she carried before entering the coma is gone. The only thing on her mind is to have revenge on the assassination team that betrayed her - a team she was once part of.]]></plotSummary>
    </movie>
</movies>
{% endhighlight %}

So that XML above is what we will be working with. A simple document
that has a series of **movie** nodes. Each **movie** node has an
attribute for the name of the movie, followed by nodes that give us a
plot summary, the year the movie was released, and a list of directors.
The following code sample will show us how we can display the number of
movies we have in the document, as well as list each movie's name,
followed by the plot. Let's see that now.  
  
{% highlight groovy %}
class groovyXmlParsing  {
    static main(args)  {
        def xmlFileContents = new File("C:\\code\\groovyXmlParsing\\movies.xml").text;
        def movies = new XmlSlurper().parseText(xmlFileContents);

        /*
         * Show how many moves we have in the document
         */
        println "Number of movies: ${movies.movie.size()}\n";

        /*
         * Display the titles and plot for each movie.
         */
        movies.movie.each { movie ->
            println "Movie: ${movie.@name}";
            println "Plot: ${movie.plotSummary.text()}";
            println "";
        };
    }
}
{% endhighlight %}

Yes... that's it! Let's break it down a bit. The first bit of code is
Groovy's way of reading a text file using the **File** class. The
next line is the fun one. Using the class **XmlSlurper** we can
parse the text into a searchable, workable XML object. And it has a cool
name!  
  
From here we display the number of movies that are in the XML document.
How? The variable **movies** represents the XML document, and more
importantly is the root node, **movies**. So with that in mind we
now know we can reference additional nodes under **movies**, and we
do this using DOT notation. **movies.movie.size()** is asking Groovy
to get all **movie** nodes that are children of the root node
**movies**. The **size** method simply returns the number of
elements that match that node's name.  
  
Ok, but how are we displaying each movie? By using Groovy's powerful
iteration constructs we can, once again, select the **movie** nodes
under root, and call the **each** method against it. The
**each** method executes a closure, passing in each matched item,
usually as a variable named **it**. I have told Groovy to use a
different variable name, however, by specifying it with **movie
->**. This means the variable **movie** will contain the current
**movie** node for each node that matches. Effectively this is a
loop (did I mention I love Groovy??).   
  
Once inside this closure we can reference attributes and child nodes of
each **movie** node that is given to us during each iteration. And
get this! To access attributes on a node, all you have to do is use DOT
notation, followed by the name of the attribute prefixed with the AT (@)
symbol! Cool!  
  
So I hope this gets some folks excited about learning more about
languages, and Groovy in particular! Cheers, and happy coding!

