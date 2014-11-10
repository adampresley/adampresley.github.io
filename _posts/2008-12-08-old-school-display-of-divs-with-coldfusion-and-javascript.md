---
layout: post
title: Old-School Display of DIVs with ColdFusion and Javascript
date: 2008-12-08 09:50:00
author: Adam Presley
status: Published
tags: development coldfusion javascript
slug: old-school-display-of-divs-with-coldfusion-and-javascript
---

Today I was asked for some Javascript that would show or hide a DIV when
selecting one or more values from a multi-select box based on if the
selected values existed in a mapping table. So in this scenario there is
one query that pulls back all possible values, and a second query that
pulls back any of those records that have been mapped to another piece
of data. When the selection occurs they wanted to only show the DIV if
the selected items had been mapped.  

Please note that the code I am showing here is not necessarily my
preferred method for doing this kind of thing, as usually I would have
the results of the mapped data come back in JSON from an AJAX call.
Since this is in a legacy application, however, we will be doing this
the old-hacky way.  

1.  Get our two pieces of data. The first is a dataset of all the
    elements we want to display and select from. The second is only
    those elements that are mapped.
2.  Present a multi-select box which an "onchange" handler pointing to a
    Javascript function. For fun we'll call that function
    "showHideMyDivy", cause we are going to name the DIV "myDivy".
3.  Create a DIV called "myDivy" and set the display style to "none".
4.  Add a function to the Array object prototype called "indexOf". Why?
    Because Firefox may have one of these functions, but IE does not. We
    need it to get the index of the array item that has the selected
    value from the select box.
5.  Create a global variable in Javascript called "mappedElements". This
    will be a Javascript array representation of the elementIds that
    come back from our second query.
6.  Create our "showHideMyDivy" function to check all selected items to
    see if they are in the "mappedElements" array. If any one of the
    selected items is in the array, show the DIV, otherwise hide it.
  
And that's the gist. Here's the code.  

    :::coldfusion
    <cfquery name="qryElements" datasource="#application.dsn#">
        SELECT elementId, elementName FROM elements
    </cfquery>

    <cfquery name="qryMappedElements" datasource="#application.dsn#">
        SELECT elementId FROM elements WHERE mappedId IS NOT NULL
    </cfquery>

    <html>
    <head>

    <script language="javascript">
        var mappedElements = [
            <cfoutput query="qryMappedElements">#qryMappedElements.elementId#<cfif qryMappedElements.currentRow LT qryMappedElements.recordCount>,</cfif></cfoutput>
        ];

        // This prototype is provided by the Mozilla foundation and
        // is distributed under the MIT license.
        // http://www.ibiblio.org/pub/Linux/LICENSES/mit.license
        // This is to support IE
        if (!Array.prototype.indexOf) {
            Array.prototype.indexOf = function(elt /*, from*/) {
                var len = this.length;

                var from = Number(arguments[1]) || 0;
                from = (from < 0) ? Math.ceil(from) : Math.floor(from);
                if (from < 0) {
                    from += len;
                }

                for (; from < len; from++) {
                    if (from in this &amp;&amp; this[from] === elt) return from;
                }

                return -1;
            };
        }

        function showHideMyDivy(obj) {
            var selectedIndex = -1; var displayFlag = false;
            var myDivyEl = document.getElementById('myDivy');
            var index = 0;

            for (index = 0; index < obj.options.length; index++) {
                var option = obj.options[index];

                selectedIndex = (option.selected &amp;&amp; mappedElements.indexOf(Number(option.value))) ? mappedElements.indexOf(Number(option.value)) : -1;
                if (selectedIndex > -1) {
                    displayFlag = true;
                    break;
                }
            }

            myDivyEl.style.display = (displayFlag) ? 'block' : 'none';
        }

    </script>
    </head>

    <body>
        <select name="selecty" id="selecty" onchange="showHideMyDivy(this);" multiple>
            <cfoutput query="qryElements">
                <option value="#qryElements.elementId#">#qryElements.elementName#</option>
            </cfoutput>
        </select>

        <div id="myDivy" style="display: none;">A cool div</div>

    </body>
    </html>

No, it's not super-sexy AJAX, but it does get the job done, and
sometimes, when working with legacy applications, that's just the way it
has to be. How would I have done this normally? I would've created a
server-side page to execute the query against the selected elements, and
returned a JSON object indicating if any of the values were mapped,
hiding or showing the DIV appropriately.
