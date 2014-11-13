---
layout: post
title: Automate Compressing JS and CSS with ANT
date: 2009-12-31 08:01:00
author: Adam Presley
status: Published
tags: development javascript ant
slug: automate-compressing-js-and-css-with-ant
---
I'm still a bit of a n00b with ANT still, but I took the time today to
improve my skills with it a tad to help automate a task which has become
quite repetitive. Once we are ready to roll changes to our application
we first have to minify our JavaScript and CSS files. We use the [YUI
Compressor](http://refresh-sf.com/yui/) which is a very nice compression
engine for both JS and CSS. It also just so happens that they have a 
Java executable JAR to run compression against your files with.  
  
As such I set out to make an ANT script. For an entry talking about the
basics of building ANT scripts see my blog entry on [Using ANT to
Automate a Build](http://blog.adampresley.com/2009/using-ant-to-automate-a-build/). 
In this post I'm simply going to cover how to call the JAR file to 
perform minification.  
  
First and foremost download the [YUI Compressor Java tool](http://yuilibrary.com/downloads/), 
and extract it to some accessible location. This is the application that
does all the magic.   
  
To make life easier I started with a few properties to make this a tad
more configurable. Below I setup properties for the client's application
directory, the path to the compressor JAR, as well as the JavaScript and
CSS directories.  
  
{% highlight xml %}
<property name="app" value="app-folder" />
<property name="compressorJar" value="\\devpath\yui-compressor\build\yuicompressor-2.4.2.jar" />
<property name="jsSourceDir" value="\\devpath\${app}\js" />
<property name="cssSourceDir" value="\\devpath\${app}\css" />
{% endhighlight %}

After this we create a target... we'll call it **minify** ( that's
original... :/ ). In this method we want to do two things: minify our
JavaScript file, and minify our CSS file. Clearly in my case we have
only two files to minify, and your mileage may vary. Let's take a look
at that.  
  
{% highlight xml %}
<java jar="${compressorJar}" fork="true" failonerror="true">
	<arg value="--line-break" />
	<arg value="4000" />
	<arg value="--type" />
	<arg value="js" />
	<arg value="--preserve-semi" />
	<arg value="-o" />
	<arg value="${jsSourceDir}\library.min.js" />
	<arg value="${jsSourceDir}\library.js" />
</java>
{% endhighlight %}

The task **java** executes the Java command line for the default
installed JVM. Notice the attributes used here. The first is *jar*. This
attribute specifies the name of the JAR file to use. We've passed the
property *compressorJar*, which as we saw above contains the full path
to our YUI compressor JAR. Also notice that to run a JAR file we need to
tell ANT to "fork" a new process. This is done by providing the *fork*
attribute and set it to true. I am also setting the *failonerror*
attribute to true so that if the JAR file returns a non-zero value the
ANT script will halt with an error.  
  
From here the YUI Compressor has a good number of arguments it can take
to customize the minifcation experience. What I've done here is tell it
to add line breaks at 4000 characters (--line-break 4000), the type of
file is a JavaScript file (--type js), to preserve excess semicolons
(--preserve-semi), and that we have an output file specified. The final
argument is the input file.  
  
After this we do the same to the CSS. Notice the lack of the
--preserve-semi argument. That argument does not apply to CSS
minification.   
  
{% highlight xml %}
<java jar="${compressorJar}" fork="true" failonerror="true">
	<arg value="--line-break" />
	<arg value="4000" />
	<arg value="--type" />
	<arg value="css" />
	<arg value="-o" />
	<arg value="${cssSourceDir}\style.min.css" />
	<arg value="${cssSourceDir}\style.css" />
</java>
{% endhighlight %}

Below is the script in its full glory. Happy coding!  
  
{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<project name="Minify JS and CSS" default="minify" basedir=".">
	<description>
		ANT build script to automate minification of library.js
		and style.css.
	</description>

	<property name="app" value="app-folder" />
	<property name="compressorJar" value="\\devpath\yui-compressor\build\yuicompressor-2.4.2.jar" />
	<property name="jsSourceDir" value="\\devpath\${app}\js" />
	<property name="cssSourceDir" value="\\devpath\${app}\css" />

	<target name="minify">
		<echo message="Starting minification process." />

		<echo message="Minifying JS..." />
		<java jar="${compressorJar}" fork="true" failonerror="true">
			<arg value="--line-break" />
			<arg value="4000" />
			<arg value="--type" />
			<arg value="js" />
			<arg value="--preserve-semi" />
			<arg value="-o" />
			<arg value="${jsSourceDir}\library.min.js" />
			<arg value="${jsSourceDir}\library.js" />
		</java>

		<echo message="Minifying CSS..." />
		<java jar="${compressorJar}" fork="true" failonerror="true">
			<arg value="--line-break" />
			<arg value="4000" />
			<arg value="--type" />
			<arg value="css" />
			<arg value="-o" />
			<arg value="${cssSourceDir}\style.min.css" />
			<arg value="${cssSourceDir}\style.css" />
		</java>
	</target>
</project>
{% endhighlight %}
