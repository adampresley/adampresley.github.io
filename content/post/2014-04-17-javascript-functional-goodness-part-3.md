---
author: "Adam Presley"
date: 2014-04-17T05:17:00Z
tags: ["development", "javascript"]
title: "JavaScript Functional Goodness Part 3"
slug: "javascript-functional-goodness-part-3"
---

More and more I am trying to stretch my "functional muscles" any time I have a chance. I've been working on an enhancement to my blog engine by adding a button to the [editor I am using](http://toopay.github.io/bootstrap-markdown/) for editing my Markdown-based posts. This new button, when clicked, will display a dialog widget with thumbnails for all images in an Amazon S3 bucket.

![S3 Browser Screenshot](http://s3.amazonaws.com/www.adampresley.com/posts/amazon-s3-browser-1.png)

<!-- excerpt -->

When the widget is initialized it makes a call to a RESTful endpoint that is responsible for authenticating credentials and retrieving bucket contents. The result of that call returns data that looks something like this.

{{< highlight javascript >}}
[
    {"url":"http://adampresley.com/image1.png", "name": "image1.png"},
    {"url":"http://adampresley.com/image2.png", "name": "image2.png"},
    {"url":"http://adampresley.com/image3.png", "name": "image3.png"},
    {"url":"http://adampresley.com/image4.png", "name": "image4.png"}
]
{{< / highlight >}}

Originally I had the code perform an AJAX call using jQuery and then did a good old-fashioned **for** loop over that data. Inside that loop I was concatenating to a string the DOM necessary to create image elements to display thumbnails of each image. Then I decided to try to clean it up and go a little functional on it.

The first order of business we must address is to define a function that will create the DOM element for a single thumbnail image given a single item in the array demonstrated above. This code creates an image element and attaches the *src* attribute which is the URL to the image. It also stores the S3 key name in the attribute *data-name* for retrieval later on. Then we setup some CSS attributes to get it sized right and return our new element. That looks like this.

{{< highlight javascript >}}
_createThumbnailItemDom = function(thumbnailItem) {
    return $("<img />")
        .attr("src", thumbnailItem.url)
        .attr("data-name", thumbnailItem.name)
        .addClass("s3BrowserWidgetItem")
        .css({
            width: "150px",
            height: "150px",
            margin: "15px"
        });
}
{{< / highlight >}}

Next up we need a method to iterate over each item in our returned dataset and craft the DOM elements, then attach them to a DIV container. This problem turned out to be a one-liner, but has two real parts to it. The first part of this is transforming the current object in our array to a DOM element using our method defined above. We can use the **$.map()** method in jQuery to apply a function to all items in our array like so.

{{< highlight javascript >}}
$.map(data, _createThumbnailItemDom)
{{< / highlight >}}

The above line of code will return a new array with each item in the old array transformed into DOM elements. Now we need to append all those elements to a target element. So what we will do is create a method that takes a reference to the target element and the array of AJAX data. We then will iterate over that dataset using jQuery's **$.each()** method. As you may recall the **$.each()** method takes two arguments. The first is an array, and the second is a function that takes an index and a current item. In this function you do whatever you wish with the item. In our case, we want to append it to the target DOM element.

{{< highlight javascript >}}
_createThumbnailsDom = function(targetEl, data) {
    $.each($.map(data, _createThumbnailItemDom), function(index, el) { targetEl.append(el); });
}
{{< / highlight >}}

The final step is to make an AJAX call to get our data, then on successful return call our **_createThumbnailsDom()** method to create our thumbnails.

{{< highlight javascript >}}
var onSuccess = function(response) { _createThumbnailsDom($(el), response.data); };
$.ajax({ url: "/some/endpoint/for/bucketdata" }).done(onSuccess);
{{< / highlight >}}

And that's it! Below is the full code for the S3 Browser widget so far. I'm sure it will change over time, but this is how it looks now. Cheers, and happy coding!

{{< highlight javascript >}}
/**
 * Class: S3Browser
 * This class provides a visual widget for viewing files in an Amazon
 * S3 bucket. A user can select images to get a full URL to the
 * S3 location. This window also allows uploading files to Amazon S3.
 *
 * This widget is based on the jQuery UI dialog widget. As such all the
 * same options available to the dialog widget are available in the S3Browser
 * widget.
 *
 * When an image is selected in the S3 Browser an event labeled
 * *s3browser-widget.select* is fired.
 *
 * Exports:
 *    $.ui.S3Browser
 *
 * RequireJS Name:
 *    s3browser-widget
 *
 * Dependencies:
 *    jquery
 *    jqueryui
 *
 * Commands:
 *    open - Opens the S3 Browser
 *    close - Closes the S3 Browser
 *
 * Example:
 *    > require(["jquery", "s3browser-widget"], function($) {
 *    >    $("#someDiv").S3Browser();
 *    >    $("#someDiv").S3Browser("open");
 *    > });
 */
define(
    [
        "jquery", "rajo.pubsub", "jqueryui"
    ],
    function($, PubSub) {
        "use strict";

        var
            /**
             * Fuction: _createDom
             * Method that creates the DOM for a single instance of this dialog widget.
             *
             * Parameters:
             *    getBucketListEndpoint - URL to the service endpoint to get the list of items in an S3 bucket
             *    dialogEl              - A reference to the dialog element to render to
             *    dialogElId            - ID of the dialog element to render to
             */
            _createDom = function(getBucketListEndpoint, dialogEl, dialogElId) {
                var
                    body = "<div class=\"s3BrowserWidgetItems\" style=\"width: 100%; height: auto;\"></div>",
                    el = "#" + dialogElId + " .s3BrowserWidgetItems",

                    onSuccess = function(response) { _createThumbnailsDom($(el), response.data); };

                $.ajax({ url: getBucketListEndpoint }).done(onSuccess);

                /*
                 * Add the initial body container to the dialog. The thumbnails
                 * are loaded via AJAX and attached via the _createThumbnailsDom method.
                 */
                dialogEl.html(body);

                /*
                 * Assign a click event handler to any element with a class of
                 * "s3BrowserWidgetItem" that is a child of our container element.
                 */
                $(el).on("click", ".s3BrowserWidgetItem", function() {
                    $(el + " .s3BrowserWidgetItem").removeClass("img-thumbnail");
                    $(this).toggleClass("img-thumbnail");
                });
            },

            /**
             * Function: _createThumbnailItemDom
             * Creates an individual thumbnail DOM item and returns it.
             *
             * Parameters:
             *    thumbnailUrl - URL to the image thumbnail
             */
            _createThumbnailItemDom = function(thumbnailItem) {
                return $("<img />")
                    .attr("src", thumbnailItem.url)
                    .attr("data-name", thumbnailItem.name)
                    .addClass("s3BrowserWidgetItem")
                    .css({
                        width: "150px",
                        height: "150px",
                        margin: "15px"
                    });
            },

            /**
             * Function: _createThumbnailsDom
             * This function creates all thumbnail DOM elements in a set of data
             * and appends them to a target DOM element. The data parameter
             * is an array of thumbnail URLs.
             *
             * Parameters:
             *    targetEl - Element to attach thumbnail DOM items to
             *    data     - Array of thumbnail URLs
             */
            _createThumbnailsDom = function(targetEl, data) {
                $.each($.map(data, _createThumbnailItemDom), function(index, el) { targetEl.append(el); });
            },

            /**
             * Function: _onDelete
             * Event handler for the *Delete* button. This will publish an event named
             * *s3browser-widget.delete* with a reference to the dialog element, the
             * URL of the image, and the S3 key name.
             */
            _onDelete = function(dialogEl) {
                var selectedImageEl = $(dialogEl).find(".s3BrowserWidgetItem.img-thumbnail");

                if (selectedImageEl.length > 0) {
                    PubSub.publish("s3browser-widget.delete", {
                        dialogEl: dialogEl,
                        imageUrl: selectedImageEl[0].src,
                        name    : selectedImageEl[0].getAttribute("data-name")
                    });
                }
            },

            /**
             * Function: _onSelect
             * Event handler for the *Select* button. This will publish an event
             * named *s3browser-widget.select* with a reference to the dialog element, the
             * URL of the image, and the S3 key name.
             */
            _onSelect = function(dialogEl) {
                var selectedImageEl = $(dialogEl).find(".s3BrowserWidgetItem.img-thumbnail");

                if (selectedImageEl.length > 0) {
                    PubSub.publish("s3browser-widget.select", {
                        dialogEl: dialogEl,
                        imageUrl: selectedImageEl[0].src,
                        name    : selectedImageEl[0].getAttribute("data-name")
                    });
                }
            },

            /**
             * Function: _onView
             * Event handler for the *View* button. This will open up the selected
             * image in a new tab/window.
             */
            _onView = function(dialogEl) {
                var selectedImageEl = $(dialogEl).find(".s3BrowserWidgetItem.img-thumbnail");

                if (selectedImageEl.length > 0) {
                    window.open(selectedImageEl[0].src);
                }
            };

        /*
         * Create the widget in the "adampresley" namespace using the
         * jQuery UI WidgetFactory.
         */
        $.widget("adampresley.S3Browser", $.ui.dialog, {
            _create: function() {
                _createDom(this.options.getBucketListEndpoint, this.element, this.element[0].id);
                this._super();
            },

            options: {
                title   : "Amazon S3 Browser",
                width   : 450,
                height  : 450,
                autoOpen: false,
                modal   : true,
                resizable: true,
                buttons : [
                    {
                        text : "Select",
                        click: function() { _onSelect(this); }
                    },
                    {
                        text : "View",
                        click: function() { _onView(this); }
                    },
                    {
                        text : "Delete",
                        click: function() { _onDelete(this); }
                    }
                ],

                getBucketListEndpoint: "/admin/ajax/s3/bucket"
            }
        });
    }
);
{{< / highlight >}}