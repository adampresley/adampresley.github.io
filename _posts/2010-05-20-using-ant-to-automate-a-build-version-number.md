---
layout: post
title: Using ANT to Automate a Build Version Number
date: 2010-05-20 10:18:00
author: Adam Presley
status: Published
tags: development ant
slug: using-ant-to-automate-a-build-version-number
---
Recently I've been writing some [Mura](http://www.getmura.com/)
plugins. One of the basic pieces of the plugin is the the
**config.xml** file which contains metadata about your plugin. One piece
of that metadata is the version number of the plugin. And every time I
made a change to the plugin, I would go and change the version number in
the **config.xml** to reflect the newest build number. So if the major
version is *1* and the minor version is *2*, and I've packaged this
plugin *4* times, the version number should be *1.2.04*.

Changing the build number manually every time became a hassle so I've
cooked up a small ANT script to automate not only the task of build
numbers, but also zipping up the plugin for deployment.
My objective here is to first update the build number in the
**config.xml** file then zip up all the files and directories minus the
Eclipse **.project** file. The first thing we will need is a
*properties* file. This will contain our build number.

{% highlight text %}
#Date last modified put here by ANT
buildNumber=04
{% endhighlight %}

The intent here is that ANT will read this, automatically increment the
value, and put that value into our **config.xml**. But how can ANT know
how to do this? Good question! This is done by replacement tokens. Let's
see that shall we?

{% highlight xml %}
<plugin>
	<name>Hello World Plugin</name>

	<!-- the package value becomes a part of the directory name where the plugin is installed. -->
	<package>SayHelloPlugin</package>
	<version>1.2.@buildNumber@</version>
	<provider>AdamPresley.com</provider>
	<providerurl>http://www.adampresley.com</providerURL>
	<category>Application</category>

	<settings />
	<eventhandlers />

	<!-- For display objects the the location attribute determines whether display objects for
		  the plugin will execute locally of globallyLocally means
		  /[siteid]/includes/plugins/[package]_[pluginID]/Globally means /plugins/[package]_[pluginID]/
	-->
	<displayobjects location="global">
		<displayobject name="Say Hello" displayobjectfile="displayObjects/sayHello.cfm" />
	</displayobjects>
</plugin>
{% endhighlight %}

Notice the *<version>* node? We have replacement token in there named
**buildNumber**. This is denoted by the @ symbol on either side of the
token. ANT has a command to find and replace that token with whatever
you wish. How? First we need to load our properties file and increment
the build number. Then we need to do the replacement. Here's how that
works.

{% highlight xml %}
<propertyfile file="build.properties">
	<entry key="buildNumber" type="int" default="0" operation="+" pattern="00" />
</propertyfile>

<echo message="Updating build number in /plugin/config.xml" />
<replace file="${buildDir}\plugin\config.xml" propertyFile="build.properties">
	<replacefilter token="@buildNumber@" property="buildNumber" />
</replace>
{% endhighlight %}

The first task, *<propertyfile>* reads the **build.properties**
file and increments the **buildNumber** variable by using the
*<entry>* task. We then use the *<replace>* task to replace that
**@buildNumber@** token we placed in the **config.xml** file. The
*<replacefilter>* task tells ANT what token to replace, and what
variable from the properties file to replace it with. Cool huh?!?

If you'd like some more info on the basics of ANT you can check out my
other posts.

- [Using ANT To Automate a Build](#post/2009/01/using-ant-to-automate-a-build)
- [Automate Compressing JS and CSS with ANT](#post/2009/12/automate-compressing-js-and-css-with-ant)

Here is a full example. Happy coding!

**build.properties**

{% highlight text %}
#Date last modified put here by ANT
buildNumber=04
{% endhighlight %}


**build.xml**

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
	<project name="Hello World Plugin" default="build" basedir=".">
		<description>
			ANT build script to build the Hello World Plugin for Mura
		</description>

		<propertyfile file="build.properties">
			<entry key="buildNumber" type="int" default="0" operation="+" pattern="00" />
		</propertyfile>

		<property name="sourceDir" location="C:\code\HelloPlugin" />
		<property name="buildDir" location="${sourceDir}\build" />

		<target name="build">
			<echo message="Build ${buildNumber}" />

			<echo message="Copying files to build folder..." />
			<mkdir dir="${buildDir}" />
			<copy todir="${buildDir}">
				<fileset dir="${sourceDir}">
					<include name="**/displayObjects/*" />
					<include name="**/plugin/*" />
					<include name="index.cfm" />
				</fileset>
			</copy>

			<echo message="Updating build number in /plugin/config.xml" />
			<replace file="${buildDir}\plugin\config.xml" propertyFile="build.properties">
				<replacefilter token="@buildNumber@" property="buildNumber" />
			</replace>

			<echo message="Zipping contents..." />
			<zip destfile="${sourceDir}\HelloPlugin.zip" basedir="${buildDir}" />

			<echo message="Cleaning up..." />
			<delete includeEmptyDirs="true">
				<fileset dir="${buildDir}" includes="**" />
			</delete>
		</target>
	</project>
{% endhighlight %}