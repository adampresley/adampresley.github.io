---
layout: post
title: Experimenting with SQL to JSON in SQL Server 2008
date: 2010-07-14 03:37:00
author: Adam Presley
status: Published
tags: development sql c#
slug: experimenting-with-sql-to-json-in-sql-server-2008
---
In the last couple of days at work my friend Adrian (@iknowkungfoo) and
I (@adampresley) have been tossing around how we can improve performance
on various portions of the application we work on. On of the trouble
areas has always been large query sets that then have to be transformed
into JSON for an AJAX response.   
  
One little feature introduced in SQL Server 2005 that I had completely
forgot about is the ability to take a given query result set and turn it
into XML. It's a very nifty feature, and once Adrian reminded me of this
some thoughts started to take shape. First let's look at how the XML
feature works. Below is a query that I ran against a test database that
contains address information. Below that is a screenshot of how the
result set looks using the XML feature.  

{% highlight sql %}
SELECT
   id
   , firstName
   , lastName
   , email
   , phone
   , postalCode
FROM address
ORDER BY
   firstName, lastName 
FOR XML AUTO, ROOT('addresses')
{% endhighlight %}

Ok, so that's pretty cool. So what I've done so far is write a .NET
assembly that I register in SQL Server, wrap a SQL function around the
.NET method, and am using this to turn the XML from the query into JSON.
For more info on creating and registering a .NET method in SQL server
see my previous post [Writing C# Functions for SQL Server and CLR
Integration](http://blog.adampresley.com/2009/writing-csharp-functions-for-sql-server-and-clr-integration/).
So without any ado here is the code for the assembly.  

{% highlight c# %}
using System;
using System.Data;
using System.Data.SqlTypes;
using System.Data.SqlClient;
using Microsoft.SqlServer.Server;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.IO;
using System.Xml;
using System.Xml.XPath;

/**
 * The JSONTransformer class provides methods to encode XML datasets
 * into JSON. These methods are intended to be used by a 
 * Microsoft SQL Server 2005 or higher.
 * @author Adam Presley
 */
public partial class JSONTransformer
{
   /**
    * This method takes an XML dataset and serializes it to a 
    * JSON string acceptable for AJAX/client use.
    * 
    * <code>
    * DECLARE @resultSetXML VARCHAR(max);
    * DECLARE @rootNodeName VARCHAR(75);
    * DECLARE @elementNodeName VARCHAR(75);
    * DECLARE @result VARCHAR(max);
    * 
    * SET @rootNodeName = 'addresses';
    * SET @elementNodeName = 'address';
    * 
    * SET @resultSetXML = (
    *     SELECT
    *         id
    *         , firstName
    *         , lastName
    *         , email
    *         , phone
    *         , postalCode
    * FROM address
    *     ORDER BY
    *         firstName, lastName 
    * FOR XML AUTO, ROOT('addresses'));
    *     
    * SELECT dbo.SQLJsonEncode(@resultSetXML, @rootNodeName, @elementNodeName) AS jsonResult;
    * </code>
    * 
    * @author Adam Presley
    * @param resultSetXML - The SQL result set serialized to XML
    * @param rootNodeName - The name of the XML root node
    * @param elementNodeName - The name of a record's XML node
    * @returns A string containing the resultset as JSON.
    */
   [SqlFunction()]
   public static string Encode(string resultSetXML, string rootNodeName, string elementNodeName) {
      XmlDocument xmlDoc = new XmlDocument();
      DataSet dataset = new DataSet();
      StringBuilder result = new StringBuilder();
      int recordCount = 0;
      int currentIndex = 0;

      /*
       * Create an XPath iterator.
       */
      XmlTextReader reader = new XmlTextReader(new StringReader(resultSetXML));
      XPathDocument xdoc = new XPathDocument(reader);
      XPathNavigator nav = xdoc.CreateNavigator();
      XPathNodeIterator iter = nav.Select(rootNodeName + "/" + elementNodeName);

      recordCount = iter.Count;
      currentIndex = 0;

      result.Append("{ \"recordcount\": \"" + recordCount.ToString() + "\", \"data\": [ ");

      while (iter.MoveNext()) {
         XPathNavigator item = iter.Current;
         result.Append("{ ");

         /*
          * If we have attributes iterate over them and add them to
          * the JSON object.
          */
         if (item.HasAttributes) {
            item.MoveToFirstAttribute();
            string line = "";

            do {
               string name = item.Name;
               string value = item.Value;

               line += "\"" + name + "\": \"" + __jsonEscape(value) + "\", ";
            } while (item.MoveToNextAttribute());

            /*
             * Remove trailing comma
             */
            line = line.Substring(0, line.Length - 2);
            result.Append(line);
         }

         result.Append(" }");
         if (currentIndex < recordCount - 1) result.Append(", ");
         currentIndex++;
      }

      result.Append(" ]}");

      reader.Close();
      return result.ToString();
   }

   /**
    * This method escapes a string in a format suitable
    * for JSON and JavaScript.
    * @author Adam Presley
    * @param data - The piece of data to JavaScript encode
    * @returns A string properly encoded for JavaScript
    */
   private static string __jsonEscape(string data) {
      string result = data;

      result = result.Replace("\"", "\\\"");

      return result;
   }
}
{% endhighlight %}

Now we can run a query or proc something like this to see JSON data.

{% highlight sql %}
DECLARE @resultSetXML VARCHAR(max);
DECLARE @rootNodeName VARCHAR(75);
DECLARE @elementNodeName VARCHAR(75);
DECLARE @result VARCHAR(max);

SET @rootNodeName = 'addresses';
SET @elementNodeName = 'address';

SET @resultSetXML = (
   SELECT
      id
      , firstName
      , lastName
      , email
      , phone
      , postalCode
   FROM address
   ORDER BY
      firstName, lastName 
   FOR XML AUTO, ROOT('addresses'));

SELECT dbo.SQLJsonEncode(@resultSetXML, @rootNodeName, @elementNodeName) AS jsonResult;
{% endhighlight %}

Pretty cool! Happy coding!
