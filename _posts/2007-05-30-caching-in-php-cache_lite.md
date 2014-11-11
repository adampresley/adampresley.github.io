---
layout: post
title: "Caching in PHP: Cache_Lite"
date: 2007-05-30 11:28:00
author: Adam Presley
status: Published
tags: development php
slug: caching-in-php-cache_lite
---

For many web application developers the concept of caching is foreign.
Most of us build our pages, query our databases, and spit out the
results. This is usually sufficient as the content is entirely dynamic.
There are times, however, when the data, although not static, doesn't
change very frequently, and can benefit from caching.  
  
Caching can come in many complex flavors, but in this blog I will talk
about a **very simple** caching mechanism accomplished by using the PEAR
library's Cache_Lite class. Cache_Lite allows you to cache some
specific piece of data, or page content, or whatever, and retrieve it
quickly for reuse.  
  
As an example let us say you have a database table with all of your
customers and information about them. This table is not likely to change
**TOO** quickly, so perhaps a web page that allows you to search and
manage these customers would benefit from caching. Let us also say that
this table is named *customers*, and you have one on 2 different
database servers, because you have several clusters. Lets also assume
you have an XML file that is the **master** list of all customers across
all clusters.  
  
If you are building a web page that can search for customer by providing
information in a search criteria, you would have to read the XML file,
and then go to the customer table for each cluster to find the
information you want. Well, perhaps there is a better way. In this
example I am using an imaginary XML file which would look like this:  
  
{% highlight xml %}
<customer>
   <customername>customerA</customerName>
   <server>DB100</server>
   <active>1</active>
</customer>
{% endhighlight %}

With this information we can then go the customer table on the database
server that each customer is on to retrieve additional information.  

{% highlight php %}
<?php

require_once("Cache/Lite.php");

$cacheId = "UniqueCacheId";
$cacheXmlGroup = "xmlGroup";
$cacheCustomerGroup = "customerGroup";

$cacheOptions = array(
   "lifeTime" => 3600,
   "automaticSerialization" => true
);

$cache = new Cache_Lite($cacheOptions);

//--------------------------------------------------------------
// Read the XML master customer file. If the data is already in the
// cache use that.
//--------------------------------------------------------------
if ($cacheData = $cache->get($cacheId, $cacheXmlGroup))
   $xmlCustomerList = $cacheData;
else {
   $xmlCustomerList = simplexml_load_file("customerXmlFile.xml");
   $cache->save($xmlCustomerList, $cacheId, $cacheXmlGroup);
}

//----------------------------------------------------------------------------
// Get the detail information from the database server/cluster this customer is on.
// Assume we are already connected to each cluster database, and get all the
// data. We'll pretend we have an array of mysql connections. If this data
// is already cached, use that.
//----------------------------------------------------------------------------
if ($cacheData = $cache->get($cacheId, $cacheCustomerGroup))
   $customerArray = $cacheData;
else {
   foreach ($connectionArray as $connection) {
      $qry = mysql_query("SELECT * FROM customers", $connection);

      while ($row = mysql_fetch_object($qry)) {
         $customerArray[$row->customerName]["additionalInfo1"] = $row->additionalInfo1;
         $customerArray[$row->customerName]["additionalInfo2"] = $row->additionalInfo2;
      }
   }

   $cache->save($customerArray, $cacheId, $cacheCustomerGroup);
}

?>
{% endhighlight %}

In this example we are loading two pieces of data. The first is an XML
file with a master customer list, containing all customers across
multiple clusters. Each cluster has a database server with a customer
table that has detailed information about customer on IT'S cluster. With
this data retrieved you can then do just about anything with that data.
Search on it... Manipulate it... Data mining... whatever.  
  
The COOL thing about this is the caching. While the cache file is still
valid the XML and customer information data is pulled from the cache
file instead of reading the XML and querying the database EACH time.
This is a significant speed boost.
