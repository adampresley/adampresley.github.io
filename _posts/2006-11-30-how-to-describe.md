---
layout: post
title: How to describe
date: 2006-11-30 17:06:00
author: Adam Presley
status: Published
tags: development
slug: how-to-describe
---
I'm having a lot of trouble really sitting down and defining my data
manipulation and translation language. I come up with several ways to
describe how to accomplish certain tasks, but then those fail to fit
another type of task. Or I may find a way to do this one more difficult
task, yet the simpler tasks do not fit into this paradigm. It's giving
me a lot of trouble.  
  
Some ideas I've come up with are more **imperative** language style, and
certainly the most flexible and powerful options. But these ideas don't
really fit with the concept of a language designed to **easily**
transform and move data. Most designs I've come up with tend to lean
toward a more **declarative** type of language, or what is known as a
**Domain specific language**. The idea fits it after all.  
  
Declarative language focuses on the result, not **HOW** to achieve the
result. An example would be HTML. You do not explicitly write code to
render output, but instead you write what the end result is to look like
using "markup".  
  
Here is an example of an idea I was tossing around for mapping columns
in a data source to a target, and a few going through **transformers**,
or filters.  

{% highlight text %}
source.differentId -\> adamImport.employee.employeeId  
source.first\_name -\> adamImport.employee.firstName  
source.phoneNumber -\> /[\^0-9]/ "" -\> /([0-9]{3})([0-9]{3})([0-9]{4})/
"(\$1) \$2-\$3" -\> source.employee.homePhone  
source.socialsecuritynumber -\> FormatSSN -\> adamImport.employee.ssn  
  
transformer FormatSSN  
begin  
/[\^0-9]/ "" -\> input  
  
if lengthof input != 9 then return ""  
  
/([0-9]{3})([0-9]{2})([0-9]{4})/ "\$1-\$2-\$3" -\> input  
end  
{% endhighlight %}

The above imaginary code describes **how** the data is transformed and
moved. There are issues with this idea however that would have to be
sorted out. Overall, however, I'm liking the idea, and the syntax.
