---
author: "Adam Presley"
comments: true
date: 2014-12-14T19:51:00Z
tags: ["development", "golang", "deployment"]
title: "Deploying My First Go Application to Heroku"
slug: "deploying-my-first-go-application-to-heroku"
---

For the last week or so I've been converting a site I manage from WordPress to a simple Go application. There are three reasons I want to do this.

1. WordPress has been extreme overkill for what I actually need
2. Performance. WordPress on a shared host is just not fast enough for what I deem to be a super simple site
3. I wanted to build something other than a server-style application in Go and I really enjoy writing Go code!

<!-- excerpt -->

With these points in mind I set out to write the site using Go, the standard template library that comes with Go's standard library, and Gorilla Mux for better routing setup. I did end up building a little bit of structure to support pre-loading layout files, and also implemented some great examples I found demonstrating how one can setup variable placeholders for insertion into a layout HTML file. But for the most part I kept the application fairly simple. I also found the need for a basic "session" management, and this is where one might think Go's standard library is lacking. It did not take long to find a simple session management library by the same people that do the Gorilla Muxer, which can be found [here](https://github.com/gorilla/sessions).

Once the app was complete and working to my satisfaction I started looking around for what options I might have to deploy this to the wild. I know I could buy a VPS, or even continue using Amazon EC2, but I found that Heroku allows Go applications with the entry-level free offering. Immediately curious I started searching for what it would take to deploy my application to Heroku. Mark McGranaghan has a great [guide to deploying your first Go app to Heroku](https://mmcgrana.github.io/2012/09/getting-started-with-go-on-heroku.html) which I followed along with. There are a couple of *gotchas* I ran into, however, and I will catalog them here in the event that anyone else having these troubles comes looking for answers.

My application crashed when trying to start, or when trying to serve the first request. Here are the items that I had to adjust to make sure my application was Go friendly. Note that this is *on top* of what Mark's article steps you through.

#### SIGINT Handler
I often put a SIGINT handler in my applications. SIGINT is a signal that a OS can send to your application that signals an interrupt. On the console CTRL+C will send your application a SIGINT. When you think about it a managed cloud environment does not need this. I noted that Heroku will happily sent a SIGKILL to my application when it was not behaving as desired.

#### PORT Environment Variable
Mark's example shows the Go application binding to an environment variable named **PORT**. I often have an optional flag to specify port, and default it to something like *8080*. Turns out that this is not optional for Heroku. It will set this environment variable at startup time and your app must respect it.

#### Binding To a Specific IP
My application was binding to **localhost** explicitly. This is a no-no with Heroku. Instead use something like this: **fmt.Sprintf(":%s", os.Getenv("PORT"))**, which will bind to available IP address. In other words, Heroku manages this, and your application needs to let it do its thing.

### Results
I am very happy with the results. Even on the free Heroku tier I'm finding the Go version of this site's requests run in half the time of the WordPress version running on shared hosting. To see it in action (for now), visit [https://clearbrookband.herokuapp.com](https://clearbrookband.herokuapp.com).

Cheers, and happy coding!
