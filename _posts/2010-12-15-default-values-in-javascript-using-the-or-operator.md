---
layout: post
title: Default Values in JavaScript Using the OR Operator
date: 2010-12-15 10:37:00
author: Adam Presley
status: Published
tags: development jquery javascript
slug: default-values-in-javascript-using-the-or-operator
---
Got a question tonight asking how to check a value in jQuery, and if it
is null or blank use a default value. Enter the "default operator", or
the **OR** (**||**) operator. In the example below the variable
**value** will be assigned the value from the text box named **textBox** if it
isn't "false", or empty. If it is we then use the string "Default baby!".

{% highlight javascript %}
<input type="text" name="textBox" id="textBox" value="" />
<input type="button" name="btn" id="btn" value="Click Me" />

<script type="text/javascript">

   $(document).ready(function() {

      $("#btn").click(function() {
         var value = ($("#textBox").val() || "Default baby!");
         alert(value);
      });   

   });

</script>
{% endhighlight %}

Happy coding!
