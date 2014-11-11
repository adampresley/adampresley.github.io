---
layout: post
title: Multiple File Uploads with ColdFusion, jQuery, and Uploadify
date: 2009-10-15 18:56:00
author: Adam Presley
status: Published
tags: development jquery coldfusion javascript uploadify
slug: multiple-file-uploads-with-coldfusion-jquery-and-uploadify
---

Today had a need to work with a page that needed to allowed the quick
upload of multiple images. I decided pretty quickly that a basic loop of
file upload text boxes would be too icky, and did a quick search for
what [jQuery](http://www.jquery.com) plugins were available. The first one on the list, and
most impressive on the demo page, was [Uploadify](http://www.uploadify.com).   
  
Although the documentation is a bit on the light side, it seems to cover
just about everything you would need to get going, and I have to say
that this is a clean little Flash uploader. So I though it would be nice
to put together a little tutorial on how to get up and running with
Uploadify quickly. For this demonstration we will be using
[ColdFusion](http://www.adobe.com/devnet/coldfusion/) on the back-end to receive our uploaded files.  
  
The first thing you will need is a recent copy of jQuery. This is a
simple matter of browsing to <http://www.jquery.com> and, on the front
page, clicking on the big Download jQuery button. I would recommend
using the non-minified version for development. Save the JS file in a
directory under your site root at **/js/jquery**.   
  
Now we need the Uploadify library. Browse to
<http://www.uploadify.com/download> and download the latest version.
This will come packaged as a ZIP file, so open it up with your favorite
ZIP utility. Extract all of the contents to **/js/uploadify** under the
root of your site.  
  
Ok, now lets get started. Let's create an **index.cfm** page. We'll
start with a basic page that will have all of our CSS and JavaScript
references, as well as a paragraph for messages, and a DIV for the
Uploadify Flash object. That looks like this.  

{% highlight cfm %}
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html>
<head>
    <meta http-equiv="content-type" content="text/html; charset=utf-8" />
    <title>Uploadify Demo</title>

    <link href="/styles/uploadify.css" rel="stylesheet" type="text/css" media="screen" />

    <script language="javascript" src="/js/jquery/jquery-1.3.2.js"></script>
    <script language="javascript" src="/js/uploadify/swfobject.js"></script>
    <script language="javascript" src="/js/uploadify/jquery.uploadify.v2.1.0.js"></script>
</head>

<body>
    <h1>Uploadify Demo</h1>

    <p>
        Please select images to upload. You may select multiple, and they will begin 
        as soon as you click Ok on the Open Dialog box.
    </p>

    <p id="message" class="messageBox" style="display: none;"></p>

    <div id="uploadify"></div>
</body>
</html>
{% endhighlight %}

There are a couple of points of interest here. First note that there is
a stylesheet to load when using Uploadify. Then you will see that we
include the jQuery library, the **swfobject.js** file, and finally the
Uploadify script. As promised we also have a paragraph for displaying
messages, which starts out not displayed. We also have a DIV named
*uploadify* for the Flash uploader object.  
  
To use Uploadify you start with what jQuery does best: selecting an
element. We then apply the uploadify method and pass in a number of
parameters to set it up. In our tutorial we want to allow multiple
uploads at one time, and we want the to automatically start. That looks
like this.  
  
{% highlight javascript %}
$('#uploadify').uploadify({
    uploader: '/js/uploadify/uploadify.swf',
    folder: '/uploads',
    cancelImg: '/js/uploadify/cancel.png',
    multi: true,
    auto: true,
    script: '/uploadify.cfm',
    fileDesc: 'Image files',
    fileExt: '*.jpg;*.jpeg;*.gif;*.png',
    onAllComplete: function(event, data) {
        $('#message').html('Files uploaded successfully.').fadeIn('slow', function() {
            setTimeout('$("#message").fadeOut("slow");', 3000); 
        });
    }
});
{% endhighlight %}

The parameters are explained as follows.  

* **uploader:** - Path to the uploadify flash file (SWF)
* **folder:** - Target folder to upload files to
* **cancelImg** - Path to the closing X image
* **multi:** - True to allow multiple uploads
* **auto:** - True to start uploading once the file is selected
* **script:** - Name of the ColdFusion script to handle the upload server-side
* **fileDesc:** - Description of the file filter list
* **fileExt:** - List of file filters. These are allowed files to select for uploading, delimited by semi-colon
* **onAllComplete:** - Callback method called by Uploadify when all uploads are complete

The neat part is the *onAllComplete*, but we'll come back to that. First
let's talk about the ColdFusion script that will handle accepting the
file. This one is simple. It just has to take the file and move it to
our directory. The directory we want will be called **/uploads**.  

{% highlight cfm %}
<cffile action="upload" filefield="form.fileData" destination="#expandPath('/')#/uploads" nameconflict="overwrite" />
{% endhighlight %}

Well that was easy! This command simply gets the uploaded file from the
FORM field *fileData*, and moves it to the **/uploads** directory, and
overwrites any existing files that may get in the way.  
  
Ok, so that's cool. But now we want to alert the user when the upload is
done and tell them all went well. To do that lets look at that callback
in the JavaScript again.  

{% highlight javascript %}
onAllComplete: function(event, data) {
    $('#message').html('Files uploaded successfully.').fadeIn('slow', function() {  setTimeout('$("#message").fadeOut("slow");', 3000); });
}
{% endhighlight %}

According to the Uploadify documentation the *onAllComplete* event takes
two parameters: *event*, and *data*. For our purposes we don't care much
about them. If you want to know what they contain use **console.log**
with Firebug to inspect them. This little block of code will select the
*message* DIV, set the HTML to a message saying all went well, then
fades it in. Notice that the second parameter of **fadeIn** is a
function, indicating that we want a callback to execute once the fade in
is complete. This function, in turn, sets a time out to select the
*message* DIV again and fade it out after 3 seconds. This will give us a
nice little message that fades in, then after 3 seconds fades away.  
  
Let's look at both pieces of code in full.  
  
### /index.cfm

{% highlight cfm %}
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html>
<head>
    <meta http-equiv="content-type" content="text/html; charset=utf-8" />
    <title>Uploadify Demo</title>

    <link href="/styles/uploadify.css" rel="stylesheet" type="text/css" media="screen" />

    <script language="javascript" src="/js/jquery/jquery-1.3.2.js"></script>
    <script language="javascript" src="/js/uploadify/swfobject.js"></script>
    <script language="javascript" src="/js/uploadify/jquery.uploadify.v2.1.0.js"></script>
</head>
<body>
    <h1>Uploadify Demo</h1>

    <p>
        Please select images to upload. You may select multiple, and they will begin 
        as soon as you click Ok on the Open Dialog box.
    </p>

    <p id="message" class="messageBox" style="display: none;"></p>
    <div id="uploadify"></div>

    <script language="javascript" type="text/javascript">
        $(document).ready(function() {
            $('#uploadify').uploadify({
                uploader: '/js/uploadify/uploadify.swf',
                folder: '/uploads',
                cancelImg: '/js/uploadify/cancel.png',
                multi: true,
                auto: true,
                script: '/uploadify.cfm',
                fileDesc: 'Image files',
                fileExt: '*.jpg;*.jpeg;*.gif;*.png',
                onAllComplete: function(event, data) {
                    $('#message').html('Files uploaded successfully.').fadeIn('slow', function() { setTimeout('$("#message").fadeOut("slow");', 3000); });
                }
            });
        });
    </script>
</body>
</html>
{% endhighlight %}

### /uploadify.cfm

{% highlight cfm %}
<cffile action="upload" filefield="form.fileData" destination="#expandPath('/')#/uploads" nameconflict="overwrite" />
{% endhighlight %}

And that's it! Happy coding!
