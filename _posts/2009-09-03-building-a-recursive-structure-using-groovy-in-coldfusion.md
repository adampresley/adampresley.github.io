---
layout: post
title: Building a Recursive Structure using Groovy in ColdFusion
date: 2009-09-03 09:55:00
author: Adam Presley
status: Published
tags: development groovy coldfusion
slug: building-a-recursive-structure-using-groovy-in-coldfusion
---

Last night I worked on a particular problem where I had a database table
that represented a hierarchy of product types that looks like this.  
  
	:::sql
	CREATE TABLE `producttype` (
		`ptype_ID` int(10) unsigned NOT NULL AUTO_INCREMENT,
		`ptype_IDParent` int(10) unsigned NOT NULL DEFAULT '0',
		`ptype_Name` varchar(50) NOT NULL,
		`ptype_ViewOrder` int(10) unsigned NOT NULL,
		`ptype_DateCreated` datetime DEFAULT NULL,
		`ptype_DateModified` datetime DEFAULT NULL,
		PRIMARY KEY (`ptype_ID`)
	) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=latin1  ROW_FORMAT=COMPACT;

With this MySQL table I needed to turn the recordset into a hierarchy of
structures in ColdFusion. I've certainly done this before, so this time
I wished to do this in Groovy through the magic of CFGroovy. Now, before
I show any code please refer to my post about adding the [arguments
scope to CFGroovy](|filename|/cfgroovy2-and-arguments-scope-in-functions.md), because I 
make use of this. Also note that I've only tested this in Railo so far, 
so I can't guarantee that it will behave on all CF platforms.  
  
Our first order of business is to get a query containing all of the
records we need, and ordered by parent and view order. This allows us to
group the structures together accordingly. We will also declare a
structure in the arguments scope to place our resulting structure into,
and call it *result*.  
  
	:::cfm
	<cfquery name="arguments.qryProductTypes" datasource="masterconfig">
		SELECT
			  pt.ptype_id
			, pt.ptype_idParent
			, pt.ptype_name
			, pt.ptype_viewOrder
			, pt.ptype_dateCreated
			, pt.ptype_dateModified
			, (SELECT COUNT(ptype_id) FROM producttype WHERE producttype.ptype_idParent=pt.ptype_id) AS childCount

		FROM producttype AS pt
		ORDER BY
			pt.ptype_idParent ASC,
			pt.ptype_viewOrder ASC
	</cfquery>

This query pulls all of the data like any other query. The most notable
part of it is the subquery that pulls a count of children for each row.
This is used by the code later to determine if we need to recursively
search for and build structures for children.  
  
From here we start Groovy script. We will need two functions. The first
function will search the query object for a value in a specific column,
and return an array of record indexes that match a criteria. The next
function will is actually responsible for building the tree structure.
Here is that code.  
  
	:::cfm
	<cfoutput>
	<g:script args="#arguments#">

		/**
		 * Searches a query object for a specific value.
		 *
		 * @author Adam Presley
		 * @param column Name of the column to search
		 * @param searchValue The value to search for
		 * @returns Array of matched query record indexes
		 */
		def searchQuery(String column, String searchValue) {
			def searchResult = []
			def match = false
			def more = true
			String value = ""

			arguments.qryProductTypes.first();
			while (more) {
				value = arguments.qryProductTypes.getString(column)
				if (arguments.qryProductTypes.wasNull()) value = ""
				if (value.toString() == searchValue.toString()) searchResult << arguments.qryProductTypes.getCurrentrow()

				more = arguments.qryProductTypes.next()
			}

			return searchResult;
		}

		/**
		 * Builds a tree-structure for the product types.
		 *
		 * @author Adam Presley
		 * @param parentId Number of the parent ID to start with
		 * @param key The structure key to start with
		 */
		def buildTree(int parentId, String key) {
			def matches = searchQuery("PTYPE_IDPARENT", parentId.toString())
			String currentKey = ""

			matches.each {
				index ->

				match = [
					PTYPE_ID: variables.qry3.getAt("PTYPE_ID", index),
					PTYPE_NAME: variables.qry3.getAt("PTYPE_NAME", index),
					CHILDCOUNT: variables.qry3.getAt("CHILDCOUNT", index),
					PTYPE_VIEWORDER: variables.qry3.getAt("PTYPE_VIEWORDER", index),
					PTYPE_DATECREATED: variables.qry3.getAt("PTYPE_DATECREATED", index),
					PTYPE_DATEMODIFIED: variables.qry3.getAt("PTYPE_DATEMODIFIED", index)
				]

				if (key.length())
					currentKey = key + "[\"" + match.get("PTYPE_ID") + "\"]"
				else
					currentKey = "[\"" + match.get("PTYPE_ID") + "\"]"

				evaluate("arguments.result" + currentKey + " = (railo.runtime.type.StructImpl) match")
				if ((int) match.get("CHILDCOUNT") > 0) {
					buildTree((int) match.get("PTYPE_ID"), currentKey)
				}
			}
		}

		buildTree(0, "");

	</g:script>
	</cfoutput>

Let's take a look at the **searchQuery** method first. This method takes
an argument for a column name, and one for the value to search for.
Using the built-in methods of a ColdFusion query object we seek back to
the first record, and begin to iterate over the recordset using the
**next()** method. From here we retrieve the value from the specified
column as a string, because that's simplest. We also make sure that if
the value retrieved was null, default to a blank string. If the value in
the column matches the search criteria, we add the current row index to
our *searchResult* array. That's pretty easy.  
  
The **buildTree** method is a little more fun. It also takes two
parameters. The first is the parent ID we wish to be working with to
build keys into our structure. The second argument is the current
structure key we are working with. The parent ID will always start with
zero to indicate we will pull base parent records. The key will start
out blank. Our code will append to the key argument while iterating over
records, and when we call **buildTree** recursively for a new branch,
key will contain the current record's ID (and any previous IDs added
recursively).   
  
The first thing that happens is we use the searchQuery method to get an
array of records that match against the passed in parentId argument. We
then iterate over the matched records using Groovy's nifty **each**
operator. At this point we craft a structure that contains all the data
elements that go into our resulting structure, such as the product type
name, ID, and more.   
  
After this we need to determine what key we are working with to add to
the resulting structure. If the key has no length, which will happen
only on the first call of **buildTree**, it will simply make the key
based on the PTYPE_ID. So at first it will look like this: *[1]*. If
**buildTree** was called during recursion, the previous key is passed
in, and we append to that. So, if our first record had a key of *[1]*,
and we are on record 2 and called buildTree recursively, the key will
now be *[1][2]*.  
  
Once we have a key, we use Groovy's abillity to evaluate code
dynamically to assign the *match* structure to the **arguments.result**
structure in the generated key. From here we use the CHILDCOUNT we got
in our subquery earlier to determine if the current record has any
children. If it does, we call **buildTree** again (thus the recursion)
passing in the current PTYPE_ID and the current generated key, starting
the process all over again!  
  
And that's about it! Happy coding you groovy peeps!  
  
(Full code here)  

	:::cfm
	<cffunction name="BuildProductTypeTree" returntype="any" access="public" output="true">
		<cfimport prefix="g" taglib="../../groovyEngine" />

		<cfquery name="arguments.qryProductTypes" datasource="masterconfig">
			SELECT
				  pt.ptype_id
				, pt.ptype_idParent
				, pt.ptype_name
				, pt.ptype_viewOrder
				, pt.ptype_dateCreated
				, pt.ptype_dateModified
				, (SELECT COUNT(ptype_id) FROM producttype WHERE producttype.ptype_idParent=pt.ptype_id) AS childCount

			FROM producttype AS pt
			ORDER BY
				pt.ptype_idParent ASC,
				pt.ptype_viewOrder ASC
		</cfquery>

		<cfset arguments.result = {} />

		<cfoutput>
		<g:script args="#arguments#">

			/**
			 * Searches a query object for a specific value.
			 *
			 * @author Adam Presley
			 * @param column Name of the column to search
			 * @param searchValue The value to search for
			 * @returns Array of matched query record indexes
			 */
			def searchQuery(String column, String searchValue) {
				def searchResult = []
				def match = false
				def more = true
				String value = ""

				arguments.qryProductTypes.first()
				while (more) {
					value = arguments.qryProductTypes.getString(column)
					if (arguments.qryProductTypes.wasNull()) value = ""
					if (value.toString() == searchValue.toString()) searchResult << arguments.qryProductTypes.getCurrentrow()

					more = arguments.qryProductTypes.next()
				}

				return searchResult;
			}

			/**
			 * Builds a tree-structure for the product types.
			 *
			 * @author Adam Presley
			 * @param parentId Number of the parent ID to start with
			 * @param key The structure key to start with
			 */
			def buildTree(int parentId, String key) {
				def matches = searchQuery("PTYPE_IDPARENT", parentId.toString())
				String currentKey = ""

				matches.each {
					index ->

					match = [
						PTYPE_ID: arguments.qryProductTypes.getAt("PTYPE_ID", index),
						PTYPE_NAME: arguments.qryProductTypes.getAt("PTYPE_NAME", index),
						CHILDCOUNT: arguments.qryProductTypes.getAt("CHILDCOUNT", index),
						PTYPE_VIEWORDER: arguments.qryProductTypes.getAt("PTYPE_VIEWORDER", index),
						PTYPE_DATECREATED: arguments.qryProductTypes.getAt("PTYPE_DATECREATED", index),
						PTYPE_DATEMODIFIED: arguments.qryProductTypes.getAt("PTYPE_DATEMODIFIED", index)
					]

					if (key.length())
						currentKey = key + "[\"" + match.get("PTYPE_ID") + "\"]"
					else
						currentKey = "[\"" + match.get("PTYPE_ID") + "\"]"

					if (server.containsKey("railo"))
						evaluate("arguments.result" + currentKey + " = (railo.runtime.type.StructImpl) match")
					else
						evaluate("arguments.result" + currentKey + " = (coldfusion.runtime.Struct) match")

					if ((int) match.get("CHILDCOUNT") > 0) {
						buildTree((int) match.get("PTYPE_ID"), currentKey)
					}
				}
			}

			buildTree(0, "")

		</g:script>
		</cfoutput>

		<cfreturn arguments.result />
	</cffunction>

	<cfset result = BuildProductTypeTree() />
