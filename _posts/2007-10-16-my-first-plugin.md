---
layout: post
title: My First Plugin
date: 2007-10-16 04:57:00
author: Adam Presley
status: Published
tags: development php plugins wordpress
slug: my-first-plugin
---

Yes, ladies and gentlemen, I have started learning how to write
WordPress plugins. My first plugin is one that removes the word
"Private: " from the beginning of blog and page titles (when they are
saved with a status of private).

The way this works is by creating a function that takes one argument. I
then used a regular expression to remove the offending words. The regex
I used was */\^(private:\\s\*)/i*and then I simply replaced any match
with a blank string, removing it from the source text. And in WordPress
there are two types of plugins: Action hooks, and Filter hooks.

In this case I used a filter hook, since filter hooks alter text coming
or going from the WordPress database. So to make my filter work you have
to use the *add_filter* function to register my function to handle the
filter hook "*the_title*". This hook alters the title of posts and
pages before they go out to the browser. Now, here's the code.

    :::php
    <?php

    /*
       Plugin Name: Hide "Private" In Title
       Plugin URI: http://blog.adampresley.com/?page_id=124
       Description: Removes the word "private:" from the beginning of blog entry and page titles.
       Version: 1.0
       Author: Adam Presley
       Author URI: http://www.adampresley.com

       Copyright 2007  Adam Presley  (email : psykoprogrammer@yahoo.com)

       This program is free software; you can redistribute it and/or modify
       it under the terms of the GNU General Public License as published by
       the Free Software Foundation; either version 2 of the License, or
       (at your option) any later version.

       This program is distributed in the hope that it will be useful,
       but WITHOUT ANY WARRANTY; without even the implied warranty of
       MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
       GNU General Public License for more details.

       You should have received a copy of the GNU General Public License
       along with this program; if not, write to the Free Software
       Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301  USA
    */

    //--------------------------------------------------------------------------
    // Name: filter_hidePrivateInTitle
    // Auth: Adam Presley
    // Desc: Removes the word "private:" from blog and page titles.
    //--------------------------------------------------------------------------
    function filter_hidePrivateInTitle($content) {
       return preg_replace('/^(private:\s*)/i', "", $content);
    }

    //---------------------------
    // Add the hook to WordPress.
    //---------------------------
    add_filter('the_title', 'filter_hidePrivateInTitle');

    ?>
