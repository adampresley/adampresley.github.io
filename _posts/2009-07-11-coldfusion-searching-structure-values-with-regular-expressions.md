---
layout: post
title: ColdFusion - Searching Structure Values with Regular Expressions
date: 2009-07-11 07:54:00
author: Adam Presley
status: Published
tags: development coldfusion java
slug: coldfusion-searching-structure-values-with-regular-expressions
---

One of my favorite ColdFusion bloggers, Mr. Ben Nadel, blogged the other
day about creating a custom tag that would [recursively traverse a
ColdFusion structure](http://www.bennadel.com/blog/1635-restructfindvalue-adding-regular-expression-searching-to-structfindvalue-.htm) 
and search for values that match a regular expression search criteria. 
I though this was a pretty cool idea and put his code in a test page to 
start testing and playing with it. After giving it a try, and pusing the limits, 
I came to the conclusion that this was a pretty cool idea, but under load the code started to run
pretty slow. In fact it made my CF server run out of memory at one point
with a very large structure (rougly 5,000 keys). Not one to back down
from a challenge, and make no mistake, getting this idea to run over a
5,000 key structure is a challenge, I decided to write a similar
function in Java to see if we could speed things up.  
  
To get started we need to cover a few basics on the
ColdFusion data types under the hood. The first item of consideration is
that ColdFusion structures are actually a Java class named
*coldfusion.runtime.Struct*, based on the class
*coldfusion.util.FastHashtable*. Next we see that ColdFusion arrays are
based on the *java.util.Vector* class. The ColdFusion classes can be
found in a JAR file named cfusion.jar in **/{CF8_INSTALL_DIR}/lib**
Our Java JAR file will go here so it can be used in ColdFusion (yes,
restart is required).  
  
The basic idea is we are going to create a method that will take a
ColdFusion structure and a regular expression search string, and it will
traverse the structure tree and return an array of structures that
contain the key, value, and the structure path to get to the values that
match your regular expression search. Here is such a piece of code.  
  
{% highlight java %}
package com.adampresley;

import java.util.*;
import java.lang.*;
import java.util.regex.*;
import coldfusion.util.*;

public class reStructTools extends Object {
    private Vector<fasthashtable> __searchValuesResult;
    private Vector<fasthashtable> __searchKeysResult;

    /**
     * Constructor.
     * @author Adam Presley
     */
    public reStructTools() {

    }

    /**
     * Searches a structure for values that match the regular expression pattern.
     * @author Adam Presley
     * @param input ColdFusion structure to search.
     * @param searchPattern Regular expression search pattern.
     * @return Vector<fasthashtable>
     */
    public Vector<fasthashtable> searchValues(FastHashtable input, String searchPattern) {
        Pattern p = Pattern.compile(searchPattern);

        this.__searchValuesResult = new Vector<fasthashtable>();
        this.__searchValues(input, p, "");

        return this.__searchValuesResult;
    }

    /**
     * Searches a structure for keys whose name matches the regular expression pattern.
     * @author Adam Presley
     * @param input ColdFusion structure to search.
     * @param searchPattern Regular expression search pattern.
     * @return Vector<fasthashtable>
     */
    public Vector<fasthashtable> searchKeys(FastHashtable input, String searchPattern) {
        Pattern p = Pattern.compile(searchPattern);

        this.__searchKeysResult = new Vector<fasthashtable>();
        this.__searchKeys(input, p, "");

        return this.__searchKeysResult;
    }

    @SuppressWarnings("unchecked")
    private void __searchValues(FastHashtable input, Pattern searchPattern, String startKey) {
        Enumeration<string> keys = (Enumeration<string>) input.keys();
        String key = "";
        String currentKey = "[\"" + startKey + "\"]";

        Object value = null;
        Object arrayValue = null;
        Matcher m = null;

        int index = 0;
        int count = 0;

        FastHashtable info = null;

        while (keys.hasMoreElements()) {
            key = keys.nextElement();
            value = input.get(key);

            currentKey = startKey + "[\"" + key + "\"]";

            // 
            // If we have a structure in our value, call back and search.
            //
            if (value.getClass().getName() == "coldfusion.runtime.Struct" || value.getClass().getName() == "coldfusion.util.FastHashtable") {
                this.__searchValues((FastHashtable) value, searchPattern, currentKey);
            }

            //
            // If our value is an array, iterate and search.
            //
            if (value.getClass().getName() == "coldfusion.runtime.Array") {
                count = ((Vector<object>) value).size();

                for (index = 0; index < count; index++) {
                    arrayValue = ((Vector<object>) value).get(index);

                    if (arrayValue.getClass().getName() == "coldfusion.runtime.Struct" || arrayValue.getClass().getName() == "coldfusion.util.FastHashtable") {
                        String tempKey = currentKey + "[" + (index + 1) + "]";
                        this.__searchValues((FastHashtable) arrayValue, searchPattern, tempKey); 
                    }
                }
            }

            //
            // Match the pattern.
            //
            if (value.getClass().getName() == "java.lang.String" || value.getClass().getName() == "java.lang.Double" || value.getClass().getName() == "java.lang.Integer") {
                m = searchPattern.matcher(value.toString());

                if (m.matches()) {
                    info = new FastHashtable();

                    info.put("key", key);
                    info.put("value", value);
                    info.put("path", currentKey);

                    this.__searchValuesResult.add(info);
                }
            }
        }
    }

    @SuppressWarnings("unchecked")
    private void __searchKeys(FastHashtable input, Pattern searchPattern, String startKey) {
        Enumeration<string> keys = (Enumeration<string>) input.keys();
        String key = "";
        String currentKey = "[\"" + startKey + "\"]";

        Object value = null;
        Object arrayValue = null;
        Matcher m = null;

        int index = 0;
        int count = 0;

        FastHashtable info = null;

        while (keys.hasMoreElements()) {
            key = keys.nextElement();
            value = input.get(key);

            currentKey = startKey + "[\"" + key + "\"]";

            // 
            // If we have a structure in our value, call back and search.
            //
            if (value.getClass().getName() == "coldfusion.runtime.Struct" || value.getClass().getName() == "coldfusion.util.FastHashtable") {
                this.__searchKeys((FastHashtable) value, searchPattern, currentKey);
            }

            //
            // If our value is an array, iterate and search.
            //
            if (value.getClass().getName() == "coldfusion.runtime.Array") {
                count = ((Vector<object>) value).size();

                for (index = 0; index < count; index++) {
                    arrayValue = ((Vector<object>) value).get(index);

                    if (arrayValue.getClass().getName() == "coldfusion.runtime.Struct" || arrayValue.getClass().getName() == "coldfusion.util.FastHashtable") {
                        String tempKey = currentKey + "[" + (index + 1) + "]";
                        this.__searchKeys((FastHashtable) arrayValue, searchPattern, tempKey); 
                    }
                }
            }

            //
            // Match the pattern.
            //
            m = searchPattern.matcher(key);

            if (m.matches()) {
                info = new FastHashtable();

                info.put("key", key);
                info.put("path", currentKey);

                this.__searchKeysResult.add(info);
            }
        }
    }
}
{% endhighlight %}

The real worker here is the method **__searchValues**. Basically it
starts by getting all of the structure's keys and iterates over them. To
get a structure path we keep up with where we are currently at in a
variable called *currentKey*. The next task is to determine if the value
of the current key is a structure, because if it is we need to call the
**__searchValues** method recursively passing the value as the next
structure to search. If the value is an array, we wish to start
iterating over this array, and once again check the value to see if it
is a structure that needs to be traversed.  
  
If the value is a simple type we want to try and match it against our
regular expression search criteria. Once we have a match we will take
the current key, value, and path information we've been tracking, put it
in a structure, and add it to the result array that will eventually get
back to the user.  
  
For a test case I put together code that creates a structure exactly
like Mr. Nadel's, and executes the Java based search method, and his CF
based search method, timing each one. Below you will find the code for this test.  

{% highlight cfm %}
<cfsetting requesttimeout="300">

<!---
    Ben Nadel's reStructFindValue method.
--->
<cffunction
    name="reStructFindValue"
    access="public"
    returntype="array"
    output="false"
    hint="I search for patterns within a given ">

    <!--- Define arguments. --->
    <cfargument
        name="target"
        type="any"
        required="true"
        hint="I am the target struct being searched."
        />

    <cfargument
        name="pattern"
        type="string"
        required="true"
        hint="I am the pattern being searched."
        />

    <cfargument
        name="scope"
        type="string"
        required="false"
        default="one"
        hint="I am the scope of the search: one or all."
        />

    <cfargument
        name="path"
        type="string"
        required="false"
        default=""
        hint="The path to the current target (for recursive calling). ** NOTE: This is used internally for recursion - this is NOT an expected argument to be passed in by the user."
        />

    <!--- Define the local scope. --->
    <cfset var local = {} />

    <!--- Create an array --->
    <cfset local.results = [] />

    <!---
        Loop over target.
        NOTE: This uses a ColdFusion custom tag that unifies
        the interface for looping over both structure and
        arrays.
        http://www.bennadel.com/go/each-iteration
    --->
    <cf_each
        item="local.item"
        collection="#arguments.target#">

        <!--- Create a variable to store the base path. --->
        <cfset local.path = arguments.path />

        <!--- Add the current key to the path. --->
        <cfset local.path &= "[ ""#local.item.key#"" ]" />

        <!--- Get a handle on the new target. --->
        <cfset local.target = local.item.value />

        <!---
            Check to see if this new target is a string (or
            if it is another complex object that we need to
            iterate over).
        --->
        <cfif isSimpleValue( local.target )>

        <!---
            Check it for the pattern match on the target
            value. For now, we are going to be using
            ColdFusion's Match() method which means a sub
            set of regular expression usage. Furthermore,
            we are going to use NoCASE for each of coding.
        --->
        <cfif arrayLen( reMatchNoCase( arguments.pattern, local.target ) )>

            <!---
                The regular expression patther was found at
                least once in the target value. This is a
                valid match. Add it to the results.
            --->
            <cfset local.result = {
                key = local.item.key,
                owner = arguments.target,
                path = local.path
            } />

            <!--- Add this result to the current results. --->
            <cfset arrayAppend( local.results, local.result ) />

        </cfif>

        <!---
            Make sure this complex nested target is one that
            we can actually iterate over (all others will be
            skipped).
        --->
        <cfelseif (
            isStruct( local.target ) ||
            isArray( local.target )
        )>

            <!---
                The nested taret is not a simple value. Therefore,
                we need to perform a depth-first, recusive search
                of it for our matching pattern.
            --->
            <cfset local.childResults = reStructFindValue(
                local.target,
                arguments.pattern,
                arguments.scope,
                local.path
            ) />

            <!---
                Add the results from our nested search to the
                current results collection.
            --->
            <cfloop
                index="local.childResult"
                array="#local.childResults#">

                <!--- Add this result to the current results. --->
                <cfset arrayAppend( local.results, local.childResult ) />

            </cfloop>

        </cfif>


        <!---
            At the end of a single iteration, let's check to see
            if we were only searching for one target. If we are,
            AND we found it, we can simply return the single
            element rather than continuing on with our recursion.
        --->
        <cfif (
            (arguments.scope eq "one") &&
            arrayLen( local.results )
        )>

            <!---
            We found at least one item - trim the results
            set in case the last iteration found more than
            one.
            --->
            <cfset local.trimmedResults = [ local.results[ 1 ] ] />

            <!--- Return the trimmed result set. --->
            <cfreturn local.trimmedResults />

        </cfif>

    </cf_each>


    <!--- Return the found results. --->
    <cfreturn local.results />
</cffunction>

<cftimer label="Getting Structure Setup" type="inline">
    <cfset myData = {
        hotGirls = [
            {
                name = "Tricia",
                hair = "Brunette"
            },
            {
                name = "Kim",
                hair = "Blonde"
            }
        ],
        athleticGirls = [
            {
                name = "Tricia",
                hair = "Brunette"
            },
            {
                name = "Jen",
                hair = "Black"
            }
        ]
    } />

    <strong>Original Structure:</strong>
    <cfdump var="#myData#" expand="true" />

</cftimer>





<cfoutput>

<!---
    Test the speed of searching values using ReStructTools
--->
<cftimer label="ReStructTools" type="inline">
    <cfset structTools = createObject("java", "com.adampresley.reStructTools").init() />

    <cfset pattern1 = "(?i)brunette|brown|black" />
    <cfset result1 = structTools.searchValues(myData, pattern1) />

    <strong>Find Values: #pattern1#</strong>
    <cfdump var="#result1#" expand="false" />
</cftimer>


<!---
    Test the speed of searching values using reStructFindValue
--->
<cftimer label="Ben Nadel's reStructFindValue" type="inline">
    <cfset pattern1 = "brunette|brown|black" />
    <cfset result2 = reStructFindValue(myData, pattern1, "all") />

    <strong>Find Values: #pattern1#</strong>
    <cfdump var="#result2#" expand="false" />
</cftimer>

</cfoutput>
{% endhighlight %}
  
Happy coding!
