---
author: "Adam Presley"
date: 2010-11-16T23:54:00Z
tags: ["development", "groovy", "coldfusion", "ant", "java"]
title: "My First Real ANT Task"
slug: "my-first-real-ant-task"
---

*Notice: The opinions expressed on this blog entry are mine alone and
do not necessarily represent the views of my employer.*

Now that the employer notice is out of the way I can actually start
talking about what I wish to talk about. At work we have a big
initiative to put CFQUERYPARAM into our SQL statements that do not have
it. Since this code is quite old a large majority of it does not make
use of this handy tag, so as you might imagine it is a pretty big task
to go over 3,000 files and validate that CFQUERYPARAM was put everywhere
it needs to be.

After the initial effort a coworker of mine created a series of regular
expressions to help validate the effort and point out any areas where we
may have done the work incorrectly. And lets be honest, staring at a
screen for 8 hours and looking for CFQUERY and variables in pound signs
can start to blur pretty quickly. Given these useful regexs I though it
would be neat to have a simple ANT script that would prompt for a
directory or file name resource and run these regexs against that
resource and create a report of any potential problem areas in an Excel
spreadsheet.

To demonstrate lets take a look at what the ANT script itself looks
like.

{{< highlight xml >}}
<!--?xml version="1.0"?-->
<project name="CFQUERYPARAM Tester" default="main" basedir=".">
   <taskdef name="queryParamChecker" classname="com.apihealthcare.opt.QueryParamChecker">

   <target name="main">
      <input message="Please provide a directory path or file path to scan:" addproperty="path" defaultvalue="${basedir}">

      <queryparamchecker path="${path}" outputfile="C:\\anttasktestresults.xlsx">
   </queryparamchecker></target>
</taskdef></project>
{{< / highlight >}}

As you can see the first thing to do is "import" a custom task called
**queryParamChecker**. This is a custom ANT task that I have written
that scans a resource for potential problems based on a set of regular
expressions. We then make use of the **input** task to prompt the user
for the resource name, a directory or file path, and then pass that to
the query param checker task.

So how does one write a custom ANT task? For this I used Groovy, so I
started with a new Groovy project in Eclipse. I then added the following
JAR files to my classpath:

* ant.jar
* commons-logging-1.1.jar
* dom4j-1.6.1.jar
* poi-3.7-beta3-20100924.jar
* poi-ooxml-3.7-beta3-20100924.jar
* poi-ooxml-schemas-3.7-beta3-20100924.jar
* xmlbeans-2.3.0.jar

These JAR files give us the Apache POI project for creating Excel
documents, as well as the necessary ANT classes for building a custom
ANT task. I then created a new package in my freshly created project and
called it **com.apihealthcare.opt**. In this package I created a new
Groovy class named **QueryParamChecker**. The first thing necessary to
creating a custom ANT task is to import the Apache ANT classes, then
extend the Task class. Your new class must override the **execute()**
method, and provide **setters** for each property your new task will
support.

{{< highlight groovy >}}
package com.apihealthcare.opt

import org.apache.tools.ant.*

class QueryParamChecker extends Task {
   private String path
   private String outputFile

   @Override
   public void execute() throws BuildException {
   }

   public void setPath(String path) {
      this.path = path
   }

   public void setOutputFile(String outputFile) {
      this.outputFile = outputFile
   }
}
{{< / highlight >}}

This is the skeleton for a custom ANT task. But I clearly wanted more
than a skeleton. I need it to check resources for errors in
CFQUERYPARAM. There are three pieces of information I used to do this.
The first is an array of regular expression object that are the meat and
potatoes of what we are trying to do here. The next is an array of file
extensions that are valid for us to check, so it contains ".cfm" and
".cfc". And finally I have an array of regular expressions that are used
to filter out any undesirable files or folders, as this application has
a number of old files that are no longer used. Here are those arrays.

{{< highlight groovy >}}
/*
 * An array of regular expressions to check files against.
 * Modify this list to change the rules yo.
 */
private def checks = [
   ~/(?i)in\s*\(\s*<cfqueryparam((?!list).)*>/,
   //~/(?i)in\s*\(\s*<cfqueryparam((?!cfsqltype).)*>/,
   ~/(?i)(#\s*cfsqltype=|#\s*maxlength=|#\s*list=|#\s*value=)/,
   ~/(?i)\sin(\s|\()[^>]*(value="#listqualify|value="#replace|value="#preservesinglequotes)/,
   ~/(?i)[^<]cfqueryparam/,
   ~/(?i)<cfqueryparam[^<>]*"\s*\/[^>]/,
   ~/(?i)<cfqueryparam\s*value=#/,
   ~/(?i)value="#dateadd/,
   ~/(?i)<cfqueryparams/,
   ~/(?i)cfsqltype=""/,
   ~/(?i)(#"list|#"value|#"cfsqltype|#"maxlength)/,
   ~/(?i)order\s*by\s*<cfqueryparam/,
   ~/(?i)(cfqueryparamvalue|cfqueryparamcfsqltype|cfqueryparammaxlength|cfqueryparamlist)/,
   ~/(?i)<cfqueryparam[^>]*(?=\/\s*"\s*>)/,
   ~/(?i)charindex\([^\)]*<cfqueryparam/,
   ~/(?i)[^<!-|<!]--[^>|-].*<cfqueryparam/,
   ~/(?i)session\.(?!(hasPermission|get|usersession|set).*)/
]

/*
 * An array of extensions that we care about. Ignore all else.
 */
private def validExtensions = [
   ".cfm",
   ".cfc"
]

/*
 * File name regex patterns to ignore.
 */
private def ignores = [
   ~/(?i)(.*?)unused_(.*)/
]
{{< / highlight >}}

From here I created two functions. The first will check a single file
for errors against the regexs, and the second will recurse a directory
structure. They are both very similar, and likely could have been
written more reusable, but for now they do the trick. Essentially these
functions will read the text from the file and compare it against each
regular expression in the **checks** array. If there are matches
they are stored in a structure and put into the **badCodeResults**
array.

After these are done the **_writeOutputFile()** method is called to
take the items in **badCodeResults** and place them into an Excel
spreadsheet. To do this you first create an **XSSFWorkbook**, an
**XSSFCreationHelper**, and an **XSSFSheet** object. The sheet
is created off the workbook; essentially it creates a new worksheet in
the workbook in Excel. The I loop over the **badCodeResults** and
create cells for the file path, the offending text, and the start and
end locations of the offending text location. To top it off I then write
the file to disk.

Below is the task in its entirety. You can also [download the soure code here](http://dl.dropbox.com/u/5726689/blog-downloads/ApiAntTasks.zip). Happy coding!

{{< highlight groovy >}}
package com.apihealthcare.opt

import org.apache.tools.ant.*
import groovy.io.FileType
import org.apache.poi.poifs.filesystem.*
import org.apache.poi.xssf.extractor.*
import org.apache.poi.xssf.usermodel.*

class QueryParamChecker extends Task {
   private String path
   private String outputFile

   /*
    * An array of regular expressions to check files against.
    * Modify this list to change the rules yo.
    */
   private def checks = [
      ~/(?i)in\s*\(\s*&lt;cfqueryparam((?!list).)*>/,
      //~/(?i)in\s*\(\s*&lt;cfqueryparam((?!cfsqltype).)*>/,
      ~/(?i)(#\s*cfsqltype=|#\s*maxlength=|#\s*list=|#\s*value=)/,
      ~/(?i)\sin(\s|\()[^>]*(value="#listqualify|value="#replace|value="#preservesinglequotes)/,
      ~/(?i)[^&lt;]cfqueryparam/,
      ~/(?i)&lt;cfqueryparam[^<>]*"\s*\/[^>]/,
      ~/(?i)&lt;cfqueryparam\s*value=#/,
      ~/(?i)value="#dateadd/,
      ~/(?i)&lt;cfqueryparams/,
      ~/(?i)cfsqltype=""/,
      ~/(?i)(#"list|#"value|#"cfsqltype|#"maxlength)/,
      ~/(?i)order\s*by\s*&lt;cfqueryparam/,
      ~/(?i)(cfqueryparamvalue|cfqueryparamcfsqltype|cfqueryparammaxlength|cfqueryparamlist)/,
      ~/(?i)&lt;cfqueryparam[^>]*(?=\/\s*"\s*>)/,
      ~/(?i)charindex\([^\)]*<cfqueryparam/,
      ~/(?i)[^&lt;!-|&lt;!]--[^>|-].*&lt;cfqueryparam/,
      ~/(?i)session\.(?!(hasPermission|get|usersession|set).*)/
   ]

   /*
    * An array of extensions that we care about. Ignore all else.
    */
   private def validExtensions = [
      ".cfm",
      ".cfc"
   ]

   /*
    * File name regex patterns to ignore.
    */
   private def ignores = [
      ~/(?i)(.*?)unused_(.*)/
   ]

   @Override
   public void execute() throws BuildException {
      def fileCheck = new File(this.path)
      def result = []

      if (fileCheck.isFile()) {
         result = _doFile()
      }
      else if (fileCheck.isDirectory()) {
         result = _doDirectory()
      }
      else
         throw new Exception("The path passed in doesn't seem to be a file or a directory!")

      _writeOutputFile(result)
   }

   public void setPath(String path) {
      this.path = path
   }

   public void setOutputFile(String outputFile) {
      this.outputFile = outputFile
   }

   private def _doFile() {
      def badCodeResults = []

      /*
       * The directory we are searching goes here!
       */
      def f = new File(this.path)
      assert f.isFile()

      def filesProcessed = 0
      def badFiles = 0

      def validFile = false
      def printed = false

      /*
       * Do we care about this particular file? If not set the
       * validFile flag to false.
       */
      validExtensions.each {
         if (f.name.endsWith(it)) validFile = true
      }

      ignores.each {
         def ignoreMe = f.name ==~ it
         if (validFile != false && ignoreMe) validFile = false
      }

      /*
       * Enter here if we care.
       */
      if (validFile) {
         filesProcessed++

         /*
          * Start looping over each regex we wish to run against this file.
          */
         checks.each { regex ->
            def matcher = f.text =~ regex
            def index = 0

            /*
             * Loop over any matches in the file.
             */
            while (matcher.find()) {
               /*
                * We have a bad code match! Put it into our results array.
                */
               if (matcher.group(0) != null && matcher.group(0) != "") {
                  if (!printed) {
                     println "File: ${f.name}..."
                     badFiles++
                  }
                  printed = true

                  badCodeResults << [
                     filePath: f.getAbsolutePath(),
                     offendingText: matcher.group(0),
                     start: matcher.start(),
                     end: matcher.end()
                  ]
               }
            }
         }

      }

      println "Processed ${filesProcessed} file(s)"
      println "${badFiles} bad file(s) found"

      badCodeResults
   }

   private def _doDirectory() {
      def badCodeResults = []

      /*
       * The directory we are searching goes here!
       */
      def rootPath = this.path
      def codeBase = new File(rootPath)
      assert codeBase.isDirectory()

      def filesProcessed = 0
      def badFiles = 0

      /*
       * Iterate over all files in our source directory.
       */
      codeBase.eachFileRecurse FileType.FILES, { f ->
      def validFile = false
      def printed = false

      /*
       * Do we care about this particular file? If not set the
       * validFile flag to false.
       */
      validExtensions.each {
         if (f.name.endsWith(it)) validFile = true
      }

      ignores.each {
         def ignoreMe = f.name ==~ it
         if (validFile != false && ignoreMe) validFile = false
      }

      /*
       * Enter here if we care.
       */
      if (validFile) {
         filesProcessed++

         /*
          * Start looping over each regex we wish to run against this file.
          */
         checks.each { regex ->
            def matcher = f.text =~ regex
            def index = 0

            /*
             * Loop over any matches in the file.
             */
            while (matcher.find()) {
               /*
                * We have a bad code match! Put it into our results array.
                */
               if (matcher.group(0) != null && matcher.group(0) != "") {
                  if (!printed) {
                     println "File: ${f.name}..."
                     badFiles++
                  }
                  printed = true

                  badCodeResults << [
                     filePath: f.getAbsolutePath() - rootPath,
                     offendingText: matcher.group(0),
                     start: matcher.start(),
                     end: matcher.end()
                  ]
               }
            }
         }

      }

      println "Processed ${filesProcessed} file(s)"
      println "${badFiles} bad file(s) found"

      badCodeResults
   }

   private def _writeOutputFile(badCodeResults) {
      /*
       * Create a workbook and worksheet.
       */
      XSSFWorkbook wb = new XSSFWorkbook()
      XSSFCreationHelper helper = wb.getCreationHelper()
      XSSFSheet sheet = wb.createSheet("Search Results")

      def rowIndex = 0

      /*
       * Loop over all our bad code results and write them to
       * rows in the Excel sheet.
       */
      badCodeResults.each {
         XSSFRow row = sheet.createRow(rowIndex++)

         row.createCell(0).setCellValue(helper.createRichTextString(it.filePath))
         row.createCell(1).setCellValue(helper.createRichTextString(it.offendingText))
         row.createCell(2).setCellValue(it.start)
         row.createCell(3).setCellValue(it.end)
      }

      /*
       * Write out the results file. Note the path.
       */
      FileOutputStream out = new FileOutputStream(this.outputFile)
      wb.write(out)
      out.close()

      println "Output results written to ${this.outputFile}"
   }
}
{{< / highlight >}}

[download the full source code]: http://dl.dropbox.com/u/5726689/blog-downloads/ApiAntTasks.zip
