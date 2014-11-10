---
layout: post
title: Jasper Reports 4 and OpenBD 1.4
date: 2011-08-08 00:42:00
author: Adam Presley
status: Published
tags: development coldfusion openbd
slug: jasper-reports-4-and-openbd-14
---

I've been playing with Jasper Reports lately as a solution for handling
both pre-build and custom reports for customers in my application. The
first hurdle to overcome was learning how to actually build a report
using the iReport application. It's not like this is hard, but more the
fact that I have zero experience in building reports using a reporting
engine. Now, after a week or two of playing with it, I have the idea of
how to build something basic, including querying for data, summing and
grouping, etc...This blog entry will **not**walk you through how to
build a report, but I will tell you how to get your OpenBD 1.4
installation ready to compile and display Jasper Report files.

The first step is to get the iReport software from [JasperForge](http://jasperforge.org/).
Install this software, as this will be what you will design your reports
with. It also happens to contain the JAR files you need to work with
Jasper in your OpenBD application. From here find those JAR files at
*{installation directory}/ireport/modules/ext*. The base files you will
need are (hopefully I don't miss any):

* jasperreports-4.0.0.jar
* jasperreports-applet-4.0.0.jar
* jasperreports-fonts-4.0.0.jar
* jasperreports-javaflow-4.0.0.jar
* groovy-all-1.7.5.jar
* iText-2.1.7.jar

Copy all the above files into your *{OpenBD App Directory}/WEB-INF/lib*
folder. You will notice that you now have **two**iText\* files. The
older version number will be the one from OpenBD, and you must delete
it.

Now that you have the JARs in place, restart your Java container (Tomcat
for me). At this point I assume you have some type of report build and
saved into your application directory with a JRXML extension. From here
I will show in code how to do four things:

1. Get a connection object from a datasource
1. Compile the report and get a print object
1. Export the report as PDF
1. Export the report as Excel

The first order of business is getting a connection object. In
ColdFusion we are accustomed to having a datasource available, but the
rest of the Java world uses JDBC connection strings and
*java.sql.Connection*objects. For this to work you will have to make
sure you have the **bluedragon**folder, which contains the Admin API,
in the root of your application, or at least mapped so you can access
its components. Let's now take a look at a function that will return a
connection object based on the active datasource defined in
**application.dsn**. Also note this assumes you have session management
turned on, and that your admin password is "password".

	:::coldfusion
	<cffunction name="getConnection" output="false">
		<cfset var admin = createObject("component", "bluedragon.adminapi.Administrator") />
		<cfset var adminDS = "" />
		<cfset var ds = "" />
		<cfset var loginResult = false />
		<cfset var connection = "" />

		<cfset loginResult = admin.login("password") />
		<cfif loginResult>
			<cfset session.auth = {
				loggedIn = true,
				password = "password"
			} />
		</cfif>

		<cfset adminDS = createObject("component", "bluedragon.adminapi.Datasource") />
		<cfset ds = adminDS.getDatasources(application.dsn) />

		<cftry>
			<cfset connection = createObject("java", "java.sql.DriverManager").getConnection(
				ds[1].hoststring,
				ds[1].username,
				ds[1].password
			) />

		<cfcatch>
			<cfdump var="#ds#" />
			<cfdump var="#cfcatch#" />
			<cfrethrow />
			<cfabort />
		</cfcatch>
		</cftry>

		<cfset admin.logout() />
		<cfreturn connection />
	</cffunction>

The first part of this function first validates the password for admin
access. From here the remainder of the administrative methods assumes a
session variable called **auth**exists, and contains the keys
**loggedIn**and **password**. From here we grab the datasource by
calling **getDatasources()**.

The next statements focus on getting a connection object by passing in
the JDBC URL, user name, and password to the **getConnection()**method
of the *java.sql.DriverManager*class in Java. If all is successful we
log out of the admin session and return the connection object.

Now we can get to the fun part. This code is pretty well documented, and
warrants little discussion. The basics are that we first get a
connection object using our fancy new function. We then compile the
JRXML file and create a print object. The print object needs the
compiled report, any arguments we wish to pass, and the connection
object.

After this I show how to export your report as either PDF or Excel,
based on the URL argument **format**. Hope you enjoy. Happy coding AND
reporting!

	:::coldfusion
	<cfparam name="url.format" default="pdf" />
	<cfset local = {} />

	<!---
		This is where you would place any arguments to send to your
		report as variables.
	--->
	<cfset local.args = {} />

	<!---
		Grab the current datasource's connection. Note that the
		function getConnection() refers to application.dsn.
		Replace as necessary.
	--->
	<cfset local.connection = getConnection() />

	<!---
		Compile the report and get a print object, filling
		it with data from the database.
	--->
	<cfset local.report = createObject("java", "net.sf.jasperreports.engine.JasperCompileManager").compileReport(
		expandPath("./InactivePeopleReport.jrxml")
	) />
	<cfset local.printObject = createObject("java", "net.sf.jasperreports.engine.JasperFillManager").fillReport(
		local.report, local.args, local.connection
	) />

	<!---
		Export in the specified format.
	--->
	<cfif url.format EQ "pdf">
		<cfset local.reportData = createObject("java", "net.sf.jasperreports.engine.JasperExportManager").exportReportToPdf(
			local.printObject
		) />
		<cfheader name="content-length" value="#arrayLen(local.reportData)#" />
		<cfcontent type="application/pdf" variable="#local.reportData#" />
	<cfelseif url.format EQ "excel">
		<cfset local.exporter = createObject("java", "net.sf.jasperreports.engine.export.JRXlsExporter").init() />
		<cfset local.baos = createObject("java", "java.io.ByteArrayOutputStream").init() />

		<cfset local.exporter.setParameter(
			createObject("java", "net.sf.jasperreports.engine.JRExporterParameter").JASPER_PRINT,
			local.printObject
		) />
		<cfset local.exporter.setParameter(
			createObject("java", "net.sf.jasperreports.engine.export.JRXlsAbstractExporterParameter").IS_DETECT_CELL_TYPE,
			true
		) />
		<cfset local.exporter.setParameter(
			createObject("java", "net.sf.jasperreports.engine.export.JRXlsAbstractExporterParameter").IS_WHITE_PAGE_BACKGROUND,
			true
		) />
		<cfset local.exporter.setParameter(
			createObject("java", "net.sf.jasperreports.engine.JRExporterParameter").OUTPUT_STREAM,
			local.baos
		) />

		<cfset local.exporter.exportReport() />
		<cfset local.reportData = local.baos.toByteArray() />

		<cfheader name="Content-Disposition" value="attachment; filename=report.xls" />
		<cfcontent type="application/excel" variable="#local.reportData#" reset="true" />
	</cfif>
	<cfabort />
