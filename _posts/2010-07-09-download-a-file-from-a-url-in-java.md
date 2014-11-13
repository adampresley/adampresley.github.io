---
layout: post
title: Download A File From a URL in Java
date: 2010-07-09 14:44:00
author: Adam Presley
status: Published
tags: development java
slug: download-a-file-from-a-url-in-java
---
I was approached tonight with a question on how one could download a
file from a specific URL in Java. I had never done this in Java before,
but I have done this very same task in C#, so I figured it couldn't be
too different of a solution. After a few minutes of digging here is what
you need.  
  
First, using the **java.net** package you will need the *URL*
class. This class allows you to open a socket connection to a specified
URL. This class also has the handy ability to open up a stream to the
newly opened connection, allowing you to use the many Java stream and
reader classes.  
  
From here we probably want to actually **save** the file that we
have a connection to and put it somewhere on our filesystem. To do this
we will use a **FileOutputStream** class in the **java.io**
package.   
  
The concept is to read a specified number of bytes from the input steam
into a buffer, write those out to our file output stream, and repeat
until there are no more bytes to read. Below is a full code sample that
connects to the Mura CMS website and download the newest version of
their awesome software, then save it to a ZIP file on your C: drive
(yes, Windows). Cheers, and happy coding!  

{% highlight java %}
package com.adampresley.examples;

import java.net.*;
import java.io.*;

public class DownloadFile {
   public static void main(String[] args) {
       try {
          /*
           * Get a connection to the URL and start up
           * a buffered reader.
           */
          long startTime = System.currentTimeMillis();

          System.out.println("Connecting to Mura site...\n");

          URL url = new URL("http://www.getmura.com/currentversion/");
          url.openConnection();
          InputStream reader = url.openStream();

          /*
           * Setup a buffered file writer to write
           * out what we read from the website.
           */
          FileOutputStream writer = new FileOutputStream("C:/mura-newest.zip");
          byte[] buffer = new byte[153600];
          int totalBytesRead = 0;
          int bytesRead = 0;

          System.out.println("Reading ZIP file 150KB blocks at a time.\n");

          while ((bytesRead = reader.read(buffer)) > 0) {  
             writer.write(buffer, 0, bytesRead);
             buffer = new byte[153600];
             totalBytesRead += bytesRead;
          }

          long endTime = System.currentTimeMillis();

          System.out.println("Done. " + (new Integer(totalBytesRead).toString()) + " bytes read (" + (new Long(endTime - startTime).toString()) + " millseconds).\n");
          writer.close();
          reader.close();
      } 
      catch (MalformedURLException e) {
         e.printStackTrace();
      }
      catch (IOException e) {
         e.printStackTrace();
      }

   }

}
{% endhighlight %}
