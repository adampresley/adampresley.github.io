---
author: "Adam Presley"
date: 2009-08-19T11:06:00Z
tags: ["development", "groovy", "coldfusion"]
title: "CFGroovy2 and Arguments Scope in Functions"
slug: "cfgroovy2-and-arguments-scope-in-functions"
---

Here lately I've been dabbling in Groovy and have started using Barney
Boisvert's excellent [CFGroovy2](http://www.barneyb.com/barneyblog/projects/cfgroovy2/)
custom tag which handles the hard part of running embedded Groovy
scripts in ColdFusion. For those that haven't seen it, here is a small
taste of what a groovy script could look like in a ColdFusion function.

{{< highlight cfm >}}
<cfcomponent>

	<cffunction name="Transform" returntype="any" access="public" output="false">
		<cfargument name="query" type="query" required="true" />
		<cfargument name="keyColumn" type="string" required="true" />

		<cfimport prefix="g" taglib="../../groovyEngine" />

		<cfoutput>
		<g:script args="#arguments#">

			result = [:]

			while (arguments.query.next()) {
				row = [:]
				cols = arguments.query.getColumnList()

				cols.each {
					columnName ->

					value = arguments.query.getString(columnName).toString()
					row.put(columnName, (arguments.query.wasNull()) ? "" : value)
				}

				result.put(arguments.query.getString(arguments.keyColumn), row)
			}

			arguments.result = result
		</g:script>
		</cfoutput>

		<cfreturn arguments.result />
	</cffunction>

</cfcomponent>
{{< / highlight >}}

The trick in this code, however, is that Barney's tag does not bind the
arguments scope to the Groovy script engine, and as a result your Groovy
scripts will not have access to the arguments scope. The reason for this
is because the arguments scope isn't a true "scope" in the sense that it
gets passed around with your page context. It doesn't. I checked. :)

To circumvent this I pass a custom argument to the tag that evaluates
and sends over the arguments structure so that the script engine can
bind to it. So, in the code below, which is Barney's code with my
additions, see lines 9, 44-46, and line 94. The changes are there and
you will see them if you compare your original script.cfm to this one.

{{< highlight cfm >}}
<cfsilent>
<cffunction name="bootstrapGroovyFromJavaLoader" access="private" output="false" returntype="any">
	<cfset var cp = listToArray(getDirectoryFromPath(getCurrentTemplatePath()) & "groovy-all-1.6.0.jar") />
	<cfset var javaLoader = createObject("component", "JavaLoader").init(cp, true) />
	<cfreturn javaLoader.create("org.codehaus.groovy.jsr223.GroovyScriptEngineImpl").init() />
</cffunction>

<cffunction name="runScript" access="private" output="false" returntype="string">
	<cfargument name="body" type="string" required="true" />
	<cfargument name="args" type="struct" required="false" />

	<cfset var engine = "" />
	<cfset var binding = "" />
	<cfset var scope = "" />
	<cfset var context = "" />
	<cfset var writer = createObject("java", "java.io.StringWriter").init() />
	<cfif NOT structKeyExists(server, "cfgroovy")>
		<cfset server["cfgroovy"] = structNew() />
		<cfset server.cfgroovy["scriptCache"] = createObject("java", "java.util.WeakHashMap").init() />
		<cfset server.cfgroovy["CompilableClass"] = getPageContext().getClass().forName("javax.script.Compilable") />
	</cfif>
	<cfif NOT structKeyExists(server.cfgroovy, "#attributes.lang#Engine")>
		<cftry>
			<cfset server.cfgroovy["#attributes.lang#Engine"] = createObject("java", "javax.script.ScriptEngineManager").getEngineByName(attributes.lang) />
		<cfcatch type="java.lang.ClassNotFoundException" />
		<cfcatch type="Railo.commons.lang.classexception" />
		<cfcatch type="Object" />
		<cfcatch type="Invalid Class" />
		</cftry>
		<cfif NOT structKeyExists(server.cfgroovy, "#attributes.lang#Engine")>
			<cfif attributes.lang EQ "groovy">
				<cfset server.cfgroovy["#attributes.lang#Engine"] = bootstrapGroovyFromJavaLoader() />
			<cfelse>
				<cfthrow type="CFGroovy.UnknownLanguageException" message="The language you specified (#attributes.lang#) is not recognized." detail="You must ensure you're running Java 1.6 or greater, and have the necessary JAR(s) on your classpath." />
			</cfif>
		</cfif>
	</cfif>
	<cfset engine = server.cfgroovy["#attributes.lang#Engine"] />
	<cfset binding = engine.createBindings() />
	<cfset binding.put("variables", attributes.variables) />
	<cfset binding.put("pageContext", getPageContext()) />
	<cfif structKeyExists(arguments, "args")>
		<cfset binding.put("arguments", arguments.args) />
	</cfif>

	<cfloop list="url,form,request,cgi,session,application,server,cluster" index="scope">
		<cfif isDefined(scope)>
			<cfset binding.put(scope, evaluate(scope)) />
		</cfif>
	</cfloop>
	<cfset body = trim(body) />
	<cfif len(body) EQ 0>
		<cfthrow type="CFGroovy.EmptyScriptException" message="You attempted to execute an empty script.  This is not allowed" detail="Groovy scripts must have at least one expression in them.  Forgetting to use &lt;cfoutput> around your &lt;g:script> tag when &lt;cfsetting enableCfOutputOnly=""true"" /> is a common cause of this problem." />
	</cfif>
	<cfset context = engine.getContext().getClass().newInstance() />
	<cfset context.setBindings(binding, context.ENGINE_SCOPE) />
	<cfset context.setWriter(writer) />
	<cfset context.setErrorWriter(writer) />
	<cfif server.cfgroovy["CompilableClass"].isAssignableFrom(engine.getClass())>
		<cfset script = server.cfgroovy.scriptCache.get(body) />
		<cfif structKeyExists(variables, "script")>
			<cfset script = script.get() />
		</cfif>
		<cfif NOT structKeyExists(variables, "script")>
			<cfset script = engine.compile(body) />
			<cfset server.cfgroovy.scriptCache.put(body, createObject("java", "java.lang.ref.WeakReference").init(script)) />
		</cfif>
		<cfset script.eval(context) />
	<cfelse>
		<cfset engine.eval(body, context) />
	</cfif>
	<cfreturn writer.toString() />
</cffunction>
<cfif thisTag.executionMode EQ "start">
	<cfparam name="attributes.language" default="groovy" />
	<cfparam name="attributes.lang" default="#attributes.language#" />
	<cfparam name="attributes.variables" default="#caller#" />
	<cfif NOT structKeyExists(attributes, "script") AND NOT thisTag.hasEndTag>
		<cfthrow type="CFGroovy.NoScriptException" message="No script was supplied <g:script> tag." detail="The <g:script> tag required either a body or a 'script' attribute contining the Groovy script to execute." />
	</cfif>
<cfelse><!--- executionMode EQ "end" --->
	<cfif NOT structKeyExists(attributes, "script")>
		<cfset attributes.script = thisTag.generatedContent />
	</cfif>
	<cfset thisTag.generatedContent = "" />
</cfif>
<!--- forgive the missing linebreak - it's to avoid an unwanted line break in the caller's output --->
</cfsilent><cfif structKeyExists(attributes, "script") AND (NOT thisTag.hasEndTag OR thisTag.executionMode EQ "end")><cfoutput><cfif structKeyExists(attributes, "args")>#runScript(attributes.script, attributes.args)#<cfelse>#runScript(attributes.script)#</cfif></cfoutput></cfif>
{{< / highlight >}}

Happy coding you Groovy people!
