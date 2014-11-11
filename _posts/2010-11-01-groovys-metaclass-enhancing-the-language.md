---
layout: post
title: Groovy's MetaClass - Enhancing the Language
date: 2010-11-01 04:19:00
author: Adam Presley
status: Published
tags: development groovy
slug: groovys-metaclass-enhancing-the-language
---

Here's a little tidbit I wanted to share which demonstrates one of the
coolest bits about Groovy. Similar to how one can use prototyping in
JavaScript to enhance existing objects Groovy allows us to modify
existing classes using the **metaClass** property. In the next
example I will demonstrate how I created a new method against the
**String** class to execute the contents of the string using the
javax scripting engine: in other words I execute Groovy code
dynamically. Let's take a look at the code first.  

{% highlight groovy %}
package com.adampresley

import javax.script.ScriptContext
import javax.script.ScriptEngine
import javax.script.ScriptEngineManager
import javax.script.ScriptEngineFactory

class Base {
   ScriptEngineManager manager = null
   ScriptEngine engine = null

   public Base() {
      manager = new ScriptEngineManager()
      engine = manager.getEngineByName("groovy")

      String.metaClass.run = { Map bindings ->
         def b = engine.createBindings()
         b.putAll(bindings)

         engine.setBindings(b, ScriptContext.ENGINE_SCOPE)
         engine.eval(delegate)
      }
   }
}
{% endhighlight %}

In our constructor you can see how we first instantiate the Groovy
script engine using the *javax.script.ScriptEngineManager* class. We
then, using the **metaClass** property, add a new method named
**run** that takes a single argument which takes a Map of parameters
to bind to the script engine. This allows us to execute the text in the
string as Groovy code, and pass in a Map of parameters that our script
can use. Let's look at a sample usage.  
  
{% highlight groovy %}
def source = [ "adam", "danable", "dood" ]
def code = "source.findAll { it.contains('da') }"

def result = code.run([ source: source ])
// assert [ "adam", "danable" ] == result
{% endhighlight %}

And that's just the surface. I barely understand how all this magic
works, but I know it has come in handy for me. Why? There are various
reasons you would want to dynamically execute code, including libraries
for other languages, DSLs from code in a database, etc... Happy coding!
