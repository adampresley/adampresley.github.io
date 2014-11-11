---
layout: post
title: Groovy Script to Generate JSON from CFSCRIPT
date: 2010-10-25 04:52:00
author: Adam Presley
status: Published
tags: development groovy
slug: groovy-script-to-generate-json-from-cfscript
---

Today I was working on a bit of code that had a big SWITCH statement in
CFSCRIPT in ColdFusion that for each case would go through the motions
of registering some input parameters, then getting a reference to a
controller, or handler, for the specified input request. That code
looked a little something like this.  

{% highlight cfm %}
switch (arguments.action) {
   case "action":
      result.controller = __controller("test.actionHandler");
      __registerParameters("param1,param2");
      break;
}
{% endhighlight %}

In an effort to change how this portion of code works I wanted to take
all of these CASE statements and turn them into a JSON structure that
looks like so.  
  
{% highlight javascript %}
{
   "action": {
      "controller": "test.actionHandler",
      "parameters": "param1,param2"
   }
}
{% endhighlight %}

This would allow me to read the file, deserialize the JSON, then
instantiate and execute the correct handler without having them defined
in code.  
  
I didn't want to hand-type all of these, however, so I opened up the
GroovyConsole and created a little script to automate this task. Here's
the code.  

{% highlight groovy %}
def results = []
def currentAction = ""
def currentController = ""
def currentParams = ""

def foundCase = false
def foundParams = false
def matcher = null

new File("C:/code/app/ControllerFactory.cfc").eachLine { line ->
   if (!foundCase) {
      if (line.contains("case \"")) {
         foundCase = true
         matcher = (line =~ /(?i)case "(.*?)"/)
         currentAction = matcher[0][1]

         println "Action: ${currentAction}"
      }
   }
   else if (foundParams) {
      if (line.contains(")")) {
         foundParams = false
      }
      else {
         matcher = line =~ /(?i)"(.*?)"/
         if (matcher.size() > 0) {
            currentParams += matcher[0][1]
         }
      }
   }
   else {
      if (line.contains("break")) {
         foundCase = false
         results << [ action: currentAction, controller: currentController, params: currentParams ]
      }
      else {
         matcher = line =~ /(?i)__controller\("(.*?)"\)/
         if (matcher.size() > 0) {
            currentController = matcher[0][1]
         }

         // single line registration
         matcher = line =~ /(?i)__registerParameters\("(.*?)"\)/
         if (matcher.size() > 0) {
            currentParams = matcher[0][1]
         }

         // Multi-line registration
         matcher = line =~ /(?i)__registerParameters\(/
         if (matcher.size() > 0) {
            foundParams = true
            currentParams = ""
         }
      }
   }
}


/*
 * Write the results out to a definition file.
 */
def out = new File("C:/definitions.json")
if (out.exists()) out.delete()

out.createNewFile()

out << "{\n"

results.eachWithIndex { actionItem, index ->
   out << "\t\"${actionItem.action}\": {\n"
   out << "\t\t\"controller\": \"${actionItem.controller}\",\n"
   out << "\t\t\"parameters\": \"${actionItem.params}\"\n"
   if (index < results.size() - 1) 
      out << "\t},\n"
   else
      out << "\t}\n"
}

out << "}"
{% endhighlight %}

With this script I read the CFC in one line at a time and when I find a
CASE statement I start looking for the controller definition and the
parameters, if any. It's mostly a lot of regexs, but overall a simple
script.
