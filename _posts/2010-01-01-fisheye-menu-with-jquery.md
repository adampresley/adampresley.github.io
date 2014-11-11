---
layout: post
title: Fisheye Menu with jQuery
date: 2010-01-01 10:33:00
author: Adam Presley
status: Published
tags: development jquery coldfusion javascript
slug: fisheye-menu-with-jquery
---

For those that have a small case of "Mac envy", this post is for you
(and me, honestly). One of the neatest things about the Mac OS user
interface is the "fisheye" menu at the bottom of the screen. You know
the one, where there is a dock bar with icons on it, and when you mouse
over the icons they grow in size, giving the "fisheye" effect. Today we
are going to look at a jQuery plugin that makes putting this kind of
menu into affect in your websites easy as pie!  
  
Stefan Petre at <http://interface.eyecon.ro> has a very nice jQuery
plugin to achieve this very effect, and its usage is pretty easy! To
begin you'll need the jQuery Fisheye plugin. I've gone and created a
minified version of the code for convenience, which may be downloaded
here.  
  
First off let's start by building the shell of an HTML page, including
the markup that will drive the menu.  

{% highlight cfm %}
<cfsetting showdebugoutput="false" />
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
<head>
    <title>Example of jQuery Fisheye</title>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta name="description" content="Example of jQuery Fisheye" />
    <meta name="keywords" content="jquery,fisheye,example,code,coldfusion" />
    <meta name="author" content="Adam Presley" />

    <script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/1.4/jquery.min.js"></script>
    <script type="text/javascript" src="fisheye.js"></script>

    <style type="text/css">
        .dockwrap { margin: 105px auto; width: 100%; background-color: #333; border: 1px solid black; }
        #dock { top: 0; width: 775px; margin: 0 auto; }
        .dock-container { position: relative; top: -8px; height: 50px; }
        a.dock-item { display: block; width: 50px; position: absolute; bottom: 0; text-align: center; text-decoration: none; color: #333; }
        .dock-item span { display: none; padding-left: 20px; }
        .dock-item img { border: 0; margin: 5px 10px 0px; width: 100%; }
    </style>
</head>

<body>
    <p>
        Below is an example of using the Fisheye Menu interface from 
        <a href="http://interface.eyecon.ro" target="_blank">http://interface.eyecon.ro</a> 
        to build a sleek, snazzy menu similar in style to the Mac OS. I've
        also provided the minified version of the Fisheye JavaScript packaged up
        in a nice neat single-JS file for you <a href="fisheye.js">here</a>. 
    </p>

    <div class="dockwrap">
        <div id="dock">
            <div class="dock-container">
                <a class="dock-item" href="#"><span>Link 1</span><img src="home_48.png" alt="Link 1" /></a> 
                <a class="dock-item" href="#"><span>Link 2</span><img src="paper_content_48.png" alt="Link 2" /></a> 
                <a class="dock-item" href="#"><span>Link 3</span><img src="tabs_48.png" alt="Link 3" /></a> 
                <a class="dock-item" href="#"><span>Link 4</span><img src="newspaper_48.png" alt="Link 4" /></a> 
                <a class="dock-item" href="#"><span>Link 5</span><img src="mail_48.png" alt="Link 5" /></a>
            </div>
        </div>
    </div>
</body>
</html>
{% endhighlight %}

As you can see the markup is pretty easy. I've setup a DIV that has a
background color to give the menu something to "sit" on. This is called
*dockwrap*. Inside of this DIV is another that is the main container
for housing the menu, followed by the DIV for the dock itself. These are
called *dock* and *dock-container*.   
  
Inside here we find a series of anchor tags with a class of
*dock-item*. In each anchor is a span which will contain the title of
the menu item, followed by the image that represents the menu item.  
  
And of course we had to include jQuery and the fisheye.js files for this
all to work, naturally. :)  
  
Now we need to actually **USE** the plugin. The jQuery method is
called *Fisheye*. It takes a number of parameters, many which I do not
totally know what they do yet, but I will talk briefly about the
important ones. Let's see that code now.  

{% highlight javascript %}
$(document).ready(function() {
    $('#dock').Fisheye({
        maxWidth: 60,
        items: 'a',
        itemsText: 'span',
        container: '.dock-container',
        itemWidth: 50,
        proximity: 60,
        alignment : 'left',
        valign: 'bottom',
        halign : 'center'
    });
});
{% endhighlight %}

The first important parameter we see here is the **items** key. This
tells the plugin what elements will actually be the menu items. We want
anchor tags to be the menu items in our case. The next would be the
**itemsText** key. This parameter specifies that the text seen over
the image will be found in a *SPAN* tag. And most important of all is
the **container** key, which will dictate what markup has the actual
fisheye menu and items we wish to render.  

Go ahead and play around with it, and you will find it remarkably easy
to use, and quite fun to put into your websites! Enjoy, and happy
coding!  

{% highlight cfm %}
<cfsetting showdebugoutput="false" />
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
<head>
    <title>Example of jQuery Fisheye</title>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta name="description" content="Example of jQuery Fisheye" />
    <meta name="keywords" content="jquery,fisheye,example,code,coldfusion" />
    <meta name="author" content="Adam Presley" />

    <script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/1.4/jquery.min.js"></script>
    <script type="text/javascript" src="fisheye.js"></script>

    <style type="text/css">
        .dockwrap { margin: 105px auto; width: 100%; background-color: #333; border: 1px solid black; }
        #dock { top: 0; width: 775px; margin: 0 auto; }
        .dock-container { position: relative; top: -8px; height: 50px; }
        a.dock-item { display: block; width: 50px; position: absolute; bottom: 0; text-align: center; text-decoration: none; color: #333; }
        .dock-item span { display: none; padding-left: 20px; }
        .dock-item img { border: 0; margin: 5px 10px 0px; width: 100%; }
    </style>
</head>

<body>
    <p>
        Below is an example of using the Fisheye Menu interface from 
        <a href="http://interface.eyecon.ro" target="_blank">http://interface.eyecon.ro</a> 
        to build a sleek, snazzy menu similar in style to the Mac OS. I've
        also provided the minified version of the Fisheye JavaScript packaged up
        in a nice neat single-JS file for you <a href="fisheye.js">here</a>. 
    </p>

    <div class="dockwrap">
        <div id="dock">
            <div class="dock-container">
                <a class="dock-item" href="#"><span>Link 1</span><img src="home_48.png" alt="Link 1" /></a> 
                <a class="dock-item" href="#"><span>Link 2</span><img src="paper_content_48.png" alt="Link 2" /></a> 
                <a class="dock-item" href="#"><span>Link 3</span><img src="tabs_48.png" alt="Link 3" /></a> 
                <a class="dock-item" href="#"><span>Link 4</span><img src="newspaper_48.png" alt="Link 4" /></a> 
                <a class="dock-item" href="#"><span>Link 5</span><img src="mail_48.png" alt="Link 5" /></a>
            </div>
        </div>
    </div>

    <script type="text/javascript">
        $(document).ready(function() {
            $('#dock').Fisheye({
                maxWidth: 60,
                items: 'a',
                itemsText: 'span',
                container: '.dock-container',
                itemWidth: 50,
                proximity: 60,
                alignment : 'left',
                valign: 'bottom',
                halign : 'center'
            });
        });
    </script>   
</body>
</html>
{% endhighlight %}
