---
layout: post
title: Experiences in Writing an Integration Service in Go
date: 2015-06-08 16:30
author: Adam Presley
tags: development golang
comments: true
---
In the last 6 months I have started working more frequently with Google's relatively new language called [Go](http://golang.org). I have written some command line tools in the form of file servers and Subversion hooks. I have written a few web based tools such as local development tools for memory metrics, code searching, and more. More recently I've started using Go professionally for projects where I felt that it would fill the need for long running daemons that provide services to other applications.

<!-- excerpt -->

The most recent project I am working on involves providing data from a 3rd party back-office system used by my client to a CMS platform that we maintain. The CMS platform drives the client's public-facing website while the back-office solution offers tools such as scheduling, booking, and more. The client only recently started moving to this system and they want the website to display some data from the back-office in an effort to drive sales. To complicate matters, however, some data that they wish to present is not stored in the back-office solution, so we decided to manage the missing data in our CMS platform. This means that now the website needs data from two very different sources. The first source of data is a SQL database which feeds the CMS platform, and subsequently the website. The second source is from an XML-based API over HTTP from the back-office solution.

The idea here is to build a microservice application which aggregates data from both the SQL and HTTP sources into a single RESTful service that the CMS can consume. The microservice provides endpoints which follow a RESTful paradigm. Requests are authenticated using JWT tokens, allowing the CMS to communicate directly from JavaScript without having to replicate the model in the CMS's C# code. From a high level this is what that looks like.

**High Level Infrastructure**
![High level Infrastructure Diagram](/assets/adampresley/images/posts/golang-integration-service-diagram.png)

**Request Process**
![Request Process](/assets/adampresley/images/posts/cms-to-microservice-request-process.png)

Now if you've done any reading across the internet lately you will find that Google's Go language can be very polarizing. There are those that love the language, and others who seem to hate it.

#### Those Who Like It

* Simplicity - Not too many baked-in language constructs, no fancy OOP ideas
* Fast Compile Time - Go just compiles fast
* Native - Compiles to a single, native binary, for fast execution and easy distribution
* Goroutines - Green threads that can communicte over channels, oftentimes avoiding mutexes and locks
* Garbage Collected - Don't have to deal with manual memory management.

#### Those Who Do Not Like It

* No Exceptions - Lots of hate around checking for error codes quite frequently
* No Generics - Some types of functions built around a specific type have to be repeated for other types, due to lack of generics
* No Classes - Go has no real concept of a class. Only attaching functions to structures, and interfaces
* Garbage Collected - Not everyone likes automatic memory management. There is a cost, and Go's garbage collector is still young

Those lists are far from complete, but do illustrate a bit about the conversations that are happening out there regarding why one may or may not use Go. In my experiences so far, though, I am falling into the category of those who enjoy writing Go. Specifically for this particular job, here are a few reasons why it has been pleasant.

* Creating an HTTP server in Go is super easy
* The 3rd party API returns XML responses. Deserializing XML in Go, so far, has been as easy as defining attributes on a structure
* The standard library for HTTP is flexible enough to setup middleware for processing requests, and as such setting JWT for authentication was a breeze
* Goroutines has allowed me to excute tasks in parallel that may otherwise be a bottleneck

I'm pretty sure that Go is not the language for everything, but for this project, and many others I'm looking at, Go has been a powerful tool in my toolbox.
