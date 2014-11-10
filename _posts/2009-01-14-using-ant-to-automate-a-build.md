---
layout: post
title: Using ANT to Automate A Build
date: 2009-01-14 06:24:00
author: Adam Presley
status: Published
tags: development programming ant
slug: using-ant-to-automate-a-build
---

Last night I attended the Dallas/Fort Worth ColdFusion User Group
meeting held at the [Paladin Consulting](http://www.paladin-inc.com/)
offices, lead here locally by Dave Shuck. There were three presentations
scheduled, though one was cancelled last minute, so we only got two.
Although both presentations were good the one I got the most use of our was Dave Shuck's
introduction to ANT scripts.

I had heard of [ANT](http://ant.apache.org/), and had already been curious about getting my
hand dirty in it to see what it could do, but have never found the time
to do it. After his presentation, however, I went home and decided it
was time to at least play with it. I did have a real-world test to try
it on. A little contract project I just worked needed a small
modification on two of the PHP files, and then I needed to redistribute
it to the client. What this involved, however, was more than just
zipping it up.

First I had to make the modifications to the code and test it. Simple
enough. Then I needed to copy only the PHP and CSS files I needed to a
temporary directory, clean out my password information from the XML
configuration files and copy THOSE over. THEN I had to regenerate my
HTML documentation using the always wonderful [Doxygen](http://www.doxygen.org),
copy all THOSE files to the temporary directory, ZIP it all up, and THEN send it
to the client. ANT to the rescue!

First things first, create an XML file. Let's call it
**example-build.xml**. This is the actual script file. The very first
things to put in there are the XML header, a project element, and a
description. That will look like this.

	:::xml
	<?xml version="1.0" encoding="UTF-8"?>
	<project name="Example Application" default="build" basedir=".">
		<description>
			ANT build script for the example application.
		</description>

Now what we want to do is consider what locations we need to work with.
Firstly we need to know where our source files are coming from, and also
where they are going to. For my needs I opted to have the end result go
into a sub-folder named **build**. Also consider I'm building
documentation HTML files using Doxygen, so I'll need to know where the
Doxygen executable is at, as well as the source and target locations for
those documents reside. We want a few *properties*then. Note that your
paths will obviously vary, and will need to be changed. That will look
like this.

	:::xml
		<!-- Global properties -->
		<property name="sourceDir" location="C:\code\example-application" />
		<property name="buildDir" location="${sourceDir}\build" />
		<property name="sourceDocDir" location="${sourceDir}\documentation\html" />
		<property name="targetDocDir" location="${buildDir}\documentation" />
		<property name="doxygen" location="C:\doxygen\doxygen-1.5.3.exe" />

In ANT doing some SET of tasks is known as a **target**. Programmers,
think of it as a function, or method. At the simplest level a target has
a name, and a series of tasks to execute. Let's start by defining the
target and telling ANT to get a current date and time stamp.

	:::Xml
		<target name="build">
			<!-- Get a current date/time stamp -->
			<tstamp />

Now we would like to copy our code and supporting files to the build
directory. In my case I wanted all the PHP files EXCEPT for one, all the
CSS files, and a clean version of the XML configuration file for our
application. The *copy* task tells ANT to copy files and/or directories.
Being versatile you can specify very exacting criteria, and in this
example we will have three include patterns, and one exclude. Let's look
at that.

	:::xml
		<!-- Copy our files to the build folder. -->
		<echo message="Copying files to build folder..." />
		<copy todir="${buildDir}">
			<fileset dir="${sourceDir}">
				<include name="*.php" />
				<include name="*.css" />
				<include name="baseSettings.xml" />
				<exclude name="dBug.php" />
			</fileset>
		</copy>

Note, however, that our application doesn't read "baseSettings.xml"
which was clearly copied to the build directory. It expects
"settings.xml". "baseSettings.xml" is the clean version. What we want to
do now is rename the file to "settings.xml" in the build directory.

	:::xml
		<!--
			Get clean copies of our application configuration files. Basically
			we have "base" versions, which are clean configuration files without
			anything but base data in it. We want to copy those to the build
			directory and then rename them to what they SHOULD be.
		-->
		<echo message="Building XML configuration files..." />
		<move file="${buildDir}\baseSettings.xml" tofile="${buildDir}\settings.xml" />

Ok, now it's time to build the documentation. In my source directory I
have a *doxyfile* that tells Doxygen how to build the documentation.
Here what we want to do is remove the existing documentation files (if
any), make a *documentation* directory in the build location and have
Doxygen build the docs in the source documentation folder. After that is
done it needs to copy those files to the target documentation folder.

	:::xml
		<!--
			Build the application documentation. My PHP apps use the JavaDoc
			syntax, and a wonderful tool called Dogygen for document generation.
			Once it is built copy the generated HTML files to the target
			documentation directory.
		-->
		<echo message="Building documentation..." />
		<delete dir="${sourceDocDir}" />
		<mkdir dir="${targetDocDir}" />
		<exec executable="${doxygen}">
			<arg value="${sourceDir}\example-application.doxyfile" />
		</exec>

		<copy todir="${targetDocDir}">
			<fileset dir="${sourceDocDir}" />
		</copy>

Now that we have all the code and documentation files necessary to give
to the client I'd like to ZIP all that up into a nice, neat, date and
time stamped file.

	:::xml
		<!-- Zip up the contents. Date and timestamp the filename. -->
		<echo message="Zipping contents..." />
		<zip destfile="${buildDir}\example-application-${DSTAMP}-${TSTAMP}.zip" basedir="${buildDir}" />

Cool, so we have a ZIP file. Let's clean up the mess. This involves
deleting all the files in the build directory EXCEPT for the ZIP file,
and deleting the target documentation directory. We'll also close off
our target and project.

	:::xml
		<!-- Clean up our mess. -->
		<echo message="Cleaning up..." />
		<delete>
			<fileset dir="${buildDir}" excludes="*.zip" />
		</delete>
		<delete dir="${targetDocDir}" />

	</target>
	</project>

And that's the basics of using ANT to automate build and deployment!
Following is the script in its entirity.

	:::xml
	<?xml version="1.0" encoding="UTF-8"?>
	<project name="Example Application" default="build" basedir=".">

		<description>
			ANT build script for the example application.
		</description>

		<!-- Global properties -->
		<property name="sourceDir" location="C:\code\example-application" />
		<property name="buildDir" location="${sourceDir}\build" />
		<property name="sourceDocDir" location="${sourceDir}\documentation\html" />
		<property name="targetDocDir" location="${buildDir}\documentation" />
		<property name="doxygen" location="C:\doxygen\doxygen-1.5.3.exe" />

		<target name="build">
			<!-- Get a current date/time stamp -->
			<tstamp />

			<!-- Copy our files to the build folder. -->
			<echo message="Copying files to build folder..." />
			<copy todir="${buildDir}">
				<fileset dir="${sourceDir}">
					<include name="*.php" />
					<include name="*.css" />
					<include name="baseSettings.xml" />
					<exclude name="dBug.php" />
				</fileset>
			</copy>

			<!--
				Get clean copies of our application configuration files. Basically
				we have "base" versions, which are clean configuration files without
				anything but base data in it. We want to copy those to the build
				directory and then rename them to what they SHOULD be.
			-->
			<echo message="Building XML configuration files..." />
			<move file="${buildDir}\baseSettings.xml" tofile="${buildDir}\settings.xml" />

			<!--
				Build the application documentation. My PHP apps use the JavaDoc
				syntax, and a wonderful tool called Dogygen for document generation.
				Once it is built copy the generated HTML files to the target
				documentation directory.
			-->
			<echo message="Building documentation..." />
			<delete dir="${sourceDocDir}" />
			<mkdir dir="${targetDocDir}" />

			<exec executable="${doxygen}">
				<arg value="${sourceDir}\example-application.doxyfile" />
			</exec>

			<copy todir="${targetDocDir}">
				<fileset dir="${sourceDocDir}" />
			</copy>

			<!-- Zip up the contents. Date and timestamp the filename. -->
			<echo message="Zipping contents..." />
			<zip destfile="${buildDir}\example-application-${DSTAMP}-${TSTAMP}.zip" basedir="${buildDir}" />

			<!-- Clean up our mess. -->
			<echo message="Cleaning up..." />
			<delete>
				<fileset dir="${buildDir}" excludes="*.zip" />
			</delete>

			<delete dir="${targetDocDir}" />
		</target>
	</project>
