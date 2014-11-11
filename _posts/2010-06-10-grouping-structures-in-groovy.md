---
layout: post
title: Grouping Structures in Groovy
date: 2010-06-10 02:31:00
author: Adam Presley
status: Published
tags: general development groovy
slug: grouping-structures-in-groovy
---

A useful tidbit for those interested in playing around with Groovy. I
had need recently to take a structure, or more Groovy-like, a Map, and
group by a particular key, and iterate over the grouped items. Turns out
to be quite easy in Groovy. Let's see an example of such a structure and
how one would *normally* iterate over it and display the data.   

{% highlight groovy %}
public def Example1() {
    def sampleData = [
        [ "name": "Adam Presley", "title": "Architect", "id": 1 ],
        [ "name": "Collin Judd", "title": "Dev II", "id": 15 ],
        [ "name": "Steve Goodly", "title": "Architect", id: 2 ],
        [ "name": "Chris Jordey", title: "Dev II", id: 3 ],
        [ "name": "Ben Nadel", title: "ACP", id: 5 ]
    ];

    /*
    * Display existing map.
    */
    println "Existing structure:";
    sampleData.each {
        println "Name: ${it.name}, Title: ${it.title}";
    }
    println "";
}
{% endhighlight %}

As you can see iteration over the structure is fairly easy. Now I want
to actually group the structure items by the person's title, iterate and
display the title, then each person who has that title. Here's how,
using the same structure as above.   

{% highlight groovy %}
/*
 * Group the data by title
 */
sampleData.groupBy {
    it.title
}.each { group -> 
    println "*GROUP ${group.key}*";

    group.value.each { item ->
        println "Name: ${item.name}, Title: ${item.title}";
    }

    println "";
}
{% endhighlight %}

When you use the **groupBy** method you return, through a closure, the
criteria to group by. Groovy then gives you a new structure, each key
being the value to group by, which is an array of the group matches.
Pretty cool!   
  
Hope you enjoyed, and happy coding!
