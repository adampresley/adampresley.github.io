---
author: "Adam Presley"
date: 2010-02-19T21:59:00Z
tags: ["development", "coldfusion", "javascript"]
title: "The Joys of NOT Using CFWDDX"
slug: "the-joys-of-not-using-cfwddx"
---

I had the pleasure of working an issue just the other day where an
internal web application didn't seem to be responding. At first I
suspected some type of bug in environment configuration, or some other
type of obscure thing. The app worked in other environments, so that was
my first suspicion. After a bit of digging I discovered this little
nugget (changed a bit to protect the innocent of course).

{{< highlight cfm >}}
<cfwddx action="cfml2js" input="#rc.query#" topLevelVariable="data" />
{{< / highlight >}}

Let me start by telling you what this function does. The CFWDDX tag,
with the action of *cfml2js* takes some object, in this case a query,
and converts it to JavaScript code. Let's see what kind of JavaScript
this produces by taking an imaginary query object that has 2 columns,
**firstName**, and **lastName**, and has two records. This is
what the JavaScript would look like on the rendered page.

{{< highlight javascript >}}
<script>

data = new WddxRecordset();
col0 = new Array();
col0[0] = "Adam";
col0[1] = "Ben";
data["firstName"] = col0;
col0 = null;
col1 = new Array();
col1[0] = "Presley";
col1[1] = "Nadel";
data["lastName"] = col1;
col1 = null;

</script>
{{< / highlight >}}

Ok, now imagine that, but a query with 123 records and 8 columns. That's
about 1,021 lines of generated JavaScript put right into the page. That
by itself isn't too terrifying, but I was still having a problem. Why?
Let's take a look.

Another bit of code in a separate JS file actually had some methods
executing when the document is ready. One of those methods relied on
this array data generated above to be present. Turns out my computer was
running **so slow** that the DOM was ready before the JS was done
parsing, so the method executed, and the array wasn't quite done,
therefore the page stopped running because the variable didn't exist.
And that was the actual error that was occurring. The variable didn't
exist.

So why is this bad, and what can we do to fix it? It's bad because of
two reasons.

1. The DOM was actually ready before the script was finished parsing. The code didn't account for slower PCs.
1. There are more modern ways of doing this.

To address this issue the first order of business was to remove the CFWDDX piece. That code is quite large,
roughly 20K of generated JavaScript. We can instead render the form,
block it, then make an AJAX call to go and get that data once the DOM is
ready. This will return a JSON response that is much lighter, weighing
in at about 11K.

A coworker asked a good question at this point, suggesting that it was
bad to go out and perform an AJAX call every time I needed this data,
because it never changes during the course of application execution.
Luckily I was a step ahead of him, as once I got the data the first
time, I store it in memory to be reused as often as needed.

Although I cannot post the actual code up here, I will provide a watered
down version of it to show how I did the AJAX call with jQuery, as well
as a quick in-memory index so I could reference the resulting JSON array
by column name. Why? Because we use ColdBox, and the query is serialized
using *serializeJson()* which makes the query into an array of arrays,
and the columns are stored in a separate array. I've blogged on this
"issue" before for those interested.

{{< highlight javascript >}}
var MyPage = function(config) {
	this.initialize = function() {
		jQuery.ajax({
			data: {
				event: "section.getData",
				method: "process",
				returnFormat: "json"
			},
			url: "coldboxproxy.cfc",
			dataType: "json",
			success: function(data) {
				__config.dataIdx = __this.buildIndex(data);
				__config.data = data;
			}
		});
	};

	this.buildIndex = function(data) {
		var i = 0;
		var result = {};

		/*
		 * The COLUMNS key contains the query's columns array.
		 */
		for (i = 0; i < data.COLUMNS.length; i++) {
			result[data.COLUMNS[i]] = i;
		}

		return result;
	};

	var __config = jQuery.extend({}, config);
	var __this = this;

	this.initialize();
};
{{< / highlight >}}

In the above code we create a container object, or **namespace** as
you may hear some people call it, which holds all the functionality for
this page. This allows me to create functions and variables that won't
step on any "global" toes.

The object automatically calls the *initialize* method, which fires
off an AJAX call to get the JSON data. I then, upon success, call a
quick function to create an "index". Normally, if I wanted to reference
*lastName* from my serialized query, I would do this.

{{< highlight javascript >}}
var lastName = __config.data[0][1];
{{< / highlight >}}

The index of 1 is the second column in the query. I wanted to reference
by the column name, however. Creating that index allows me to do this.

{{< highlight javascript >}}
var lastName = __config.data[0][__config.dataIdx.LASTNAME];
{{< / highlight >}}

Although I prefer to create code on the ColdFusion side to transform the
query to JSON that allows me to do that in JavaScript, I did not have
that luxury at this point, so this was a close second.

Hope this rambling made some sense!! Happy coding!
