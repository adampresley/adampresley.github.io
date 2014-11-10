---
layout: post
title: Creating a SMTP Mail Server for Development
date: 2010-07-20 01:48:00
author: Adam Presley
status: Published
tags: development java
slug: creating-a-smtp-mail-server-for-development
---

While working on ColdFusion applications with Railo on Tomcat I have
been piecing together tools necessary to get these applications to do
everything I need them to do. So I've bolted on Hibernate, a URL
rewriter, and so on. Now I found myself needing an SMTP server of some
type so that emails generated by the application don't create a
ColdFusion error. I then thought to myself, "Wouldn't it be fun to write
a small SMTP server?" So I did.  
  
The design objectives behind this little server are simple. Accept
incoming SMTP requests on port 25, respond accordingly, and throw away
the mail. I'm not even concerned about actually relaying or sending the
message. I just need to make sure my ColdFusion application is satisfied
that a mail was given to a mail server.   
  
So what we are going to talk about in this blog entry is the basics of
how to do this in Java. In a future post I'll look at integrating this
into Tomcat so the SMTP server starts with Tomcat. The tools I will use
in this blog entry are simple: Eclipse.  
  
To get started create a new Java Project in Eclipse. I called mine
**TinyMailServer**, and I used the Java JRE version 1.6. From here
your code will need a package to live in. For mine I used
**com.adampresley.TinyMailServer**.  
  
There are three classes we are going to create: *Main*,
*SmtpServer*, and *SmtpExecutor*. Let's start with Main. Right click
on your Java package and choose New -> Class. In the field labeled
*Name* type in "Main". Also check the checkbox labeled "public static
void main(String[] args)". This will make an entry point for our
executable Java application.  
  
Now we'll need two more classes, so do the right click thing again. This
time we will **not** check the checkbox for making a Main method.
The two classes will be named *SmtpServer* and *SmtpExecutor*.   
  
Let's start with the *SmtpServer* class. This class will be
instantiated by our Main method and actually binds to port 25 for
incoming connections. We also want this to execute in a separate thread.
As such we will need to make sure the class extends the *Thread*
class.   
  
When building servers you must have your application bind to a socket at
an address on a port. To do this in Java you use the
*java.net.ServerSocket* class, which provides an easy way to bind to
the address and port as well as listen for incoming connections. So this
class we are building will create the socket object in the constructor,
and then will start our thread. When the thread is started the
**run()** method is executed, and this method will actually do the
dirty work of listening for connections on port 25, and when a
connection is received will actually do something. Let's see that code.  
  
    :::java
    package com.adampresley.TinyMailServer;

    import java.net.*;
    import java.io.*;

    public class SmtpServer extends Thread {
     private String address;
     private int port;
     private int maxRequestsInQueue;

     private ServerSocket connection;
     private Logger log;

     public SmtpServer(String Address, int Port, int MaxRequestsInQueue) throws UnknownHostException, IOException {
        address = Address;
        port = Port;
        maxRequestsInQueue = MaxRequestsInQueue;

        /*
         * Prepare to bind to an address and port.
         */
        connection = new ServerSocket(port, maxRequestsInQueue, InetAddress.getByName(address));

        /*
         * Start our thread's processing.
         */
        start();
     }

     @Override
     public void run() {
        try {
           System.out.println("SMTP Listener started.");

           do {
              /*
               * Attempt to start listening for incoming connections.
               */
              try {
                 System.out.println("Accepting connections.");
                 Socket socket = connection.accept();

                 Thread executor = new SmtpExecutor(socket, log);
                 executor.start();
              }
              catch (IOException e) {
                 System.out.format("An exception occured while attempting to listen for connections: %s", e.getMessage());
                 e.printStackTrace();
                 break;
              }

           } while (true);
        }
        catch (Throwable e) {
           System.out.format("An exception occured during the listener loop: %s", e.getMessage());
        }
     }
    }

In the code above there is quite a bit of error handling, stack dumping,
etc... The most interesting part is the loop. In this loop we call the
**accept()** method on the *ServerSocket* class. This method stops
the thread and waits for someone to talk to it. So when a connection
comes in on port 25 the connection is accepted and execution resumes.   
  
This is where our next class, *SmtpExecutor*, comes into play. Once
the connection is accepted we instantiate an instance of this class and
tell it to start. An since the *SmtpExecutor* class extends the
*Thread* class it starts a new thread, and execution of this listener
loop continues, waiting for the next connection.  
  
Now let's look at the *SmtpExecutor* class. This is the one that does
the bulk of the work. First off it extends *Thread* so we can handle
incoming connections in new threads. As such this class must override
the **run()** method, just like the *SmtpServer* class did.   
  
Before we do that, however, the constructor does a little work. The
*SmtpServer* class, when it instantiates the *SmtpExecutor* class,
passes in a copy of the socket object that has the connection. With this
we need to get a reader and writer connected to the socket's in and out
streams. This allows us to read the incoming request, and write
responses back out to the socket.   
  
So once the thread starts running the first objective is to tell the
client connecting to our socket that we are ready for additional
commands. We do this by sending "220 SMTP Server TinyMailServer ready
and waiting." to the output stream. This signals to the client that we
can receive more commands.  
  
From here we will do a loop, reading in data form the input stream,
until we receive the final command, "**QUIT**". But I'm jumping
ahead a bit. Here's how an SMTP server works.  

* A connection is made by the client to the server
* A status code of 220 signals to the client we are ready for more commands
* Commands are sent by the client to the server, each in the form of **COMMAND: data**, followed by return
* Commands are acted upon, and responses for each sent
* Commands are processed until we receive the QUIT command. Send a status code of 221 and break the loop
* Close the socket connection
  
There are a number of commands to process, and I only cover eight. The
only ones that deserve special attention for my purposes are the DATA
and QUIT commands. The QUIT command because we need to stop processing
and close the socket. The DATA command indicates that we are going to
get the mail headers and data. For this one we send another status code,
354, and tell the client how to terminate the data transmission by
sending "End with <something>". For us the something will be a
period.  
  
After that status is sent we will read from the input stream until the
period is sent. Once the terminating period is sent we send a status 250
to indicate we are ready for the next command.  
  
Whew, that's a lot to cover! Let's see that code now!  
  
    :::java
    package com.adampresley.TinyMailServer;

    import java.io.BufferedReader;
    import java.io.IOException;
    import java.io.InputStreamReader;
    import java.io.OutputStreamWriter;
    import java.io.PrintWriter;
    import java.net.Socket;
    import java.util.StringTokenizer;

    public class SmtpExecutor extends Thread {
     private Socket __socket;

     private BufferedReader __inStream;
     private PrintWriter __outStream;

     public SmtpExecutor(Socket socket) throws IOException {
        __socket = socket;
        __inStream = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        __outStream = new PrintWriter(new OutputStreamWriter(socket.getOutputStream()));
     }

     @Override
     public void run() {
        println("220 SMTP Server TinyMailServer ready and waiting.");

        try {
           do {
              String input = "";

              try {
                 input = __inStream.readLine();

                 if (input == null)
                    break;
              }
              catch (Exception e) {
                 e.printStackTrace();
              }

              /*
               * Parse commands. For now we just ignore the command
               * and say all is ok.
               */
              StringTokenizer tokenizer = new StringTokenizer(input, " :");
              String command = "";

              command = tokenizer.nextToken().toUpperCase();

              if (command.compareTo("DATA") == 0) {
                 doCommand_DATA();
                 continue;
              }

              if (command.compareTo("RCPT") == 0) {
                 println("250 OK, RCPT received");
                 continue;
              }

              if ((command.compareTo("MAIL") == 0) || (command.compareTo("SEND") == 0)) {
                 println("250 OK, MAIL/SEND received");
                 continue;
              }

              if ((command.compareTo("HELO") == 0) || (command.compareTo("EHLO") == 0)) {
                 println ("250 OK, Howdy ya'll!");
                 continue;
              }

              if (command.compareTo("RSET") == 0) {
                 println("250 OK, RSET received");
                 continue;
              }

              if (command.compareTo("QUIT") == 0) {
                 println("221 SMTP Server TinyMailServer closing transmission channel");
                 break;
              }
           } while (true);
        }
        catch (Exception e) {
           e.printStackTrace();
        }
        finally {
           try {
              __socket.close();
           }
           catch (Exception e) {
           }
        }
     }

     private void println(String s) {
        __outStream.println(s);
        __outStream.flush();

        System.out.println(s);
     }

     private void doCommand_DATA() throws IOException {
        println("354 Send me data yo. End with .");
        System.out.println("Mail:");

        do {
           String input = __inStream.readLine();
           if (input.equals(".")) {
              println("250 OK");
              break;
           }
           else {
              System.out.println(input);
           }
        } while (true);
     }
    }

Ok, so we've got a server to listen for connections. When a connection
is received we fire off a thread that actually response to the client,
then terminates the socket connection. Now we need to tie it together in
the *Main* class. This part is easy. Instantiate and run. :) Let's see
it.
  
    :::java
    package com.adampresley.TinyMailServer;

    import java.io.IOException;
    import java.net.UnknownHostException;

    public class Main {
     public static void main(String[] args) throws UnknownHostException, IOException {
        String address = "localhost";
        int port = 25;
        int maxRequestsInQueue = 10;

        try {
           System.out.format("Starting SMTP server at %s on port %s\n\n", address, port);
           System.out.println("Created by Adam Presley (c) 2010");
           System.out.println("Visit http://www.adampresley.com!!\n");
           SmtpServer server = new SmtpServer(address, port, maxRequestsInQueue);

           server.start();
        }
        catch (Exception e) {

        }
     }
    }

Now you are ready to run that thing! Go ahead, run it. To test it, create a 
ColdFusion page with the following code, then run it.  
  
    :::cfm
    <cfmail from="from@address.com" to="to@address.com" subject="Test Mail Server">Test</cfmail>

Ok, so why reinvent the wheel? Cause sometimes it's fun to. Happy
coding!