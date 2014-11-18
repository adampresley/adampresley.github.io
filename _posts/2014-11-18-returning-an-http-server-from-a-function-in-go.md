---
layout: post
title: Returning an HTTP Server From a Function in Go
date: 2014-11-18 00:14
author: Adam Presley
tags: development golang
comments: true
---
When building web applications in Go the most common example is one that sets up a handler for a specific URL route (endpoint) then fires up the HTTP server to listen for requests. It looks like this.

{% highlight go %}
package main

import (
    "net/http"

    "github.com/adampresley/myproject/controllers/versionController"
)

func main() {
    http.HandleFunc("/", versionController.GetVersion)
    http.ListenAndServe("0.0.0.0:8080", nil)
}
{% endhighlight %}

The above example shows the a basic case of listening on all IP address on port 8080, using the default *ServeMux* to handle routes. This is all well good, until the point where you may need more flexibility. I have reached that point.

Now I want to build a package that sets up my routes and returns a *Server* object that I can use in more than one project. With this I can create multiple projects that listen and serve up the same web content and write that code that serves it all up just once. To do this you need a function that can return an *Http.Server* structure, with which you can then listen to requests with. Here's how that works using [Gorilla Mux](https://github.com/gorilla/mux) and the basic Go standard library.

{% highlight go %}
import (
    "fmt"
    "net/http"

    "github.com/adampresley/myproject/controllers/versionController"

    "github.com/gorilla/mux"
)

func setupHttpRouter() http.Handler {
    router := mux.NewRouter()
    router.HandleFunc("/", versionController.GetVersion).Methods("GET", "OPTIONS")

    return router
}

func NewHttpServer(address string, port int) *http.Server {
    router := setupHttpRouter()
    listener := &http.Server{
        Addr:    fmt.Sprintf("%s:%d", address, port),
        Handler: router,
    }

    return listener
}
{% endhighlight %}

This code gives us a method named **NewHttpServer** which returns an HTTP server ready to serve requests on an address and port you specify. Now I can use this in my separate applications. In my case I wish to distribute a SOA web server app to be used in different front-end applications. An example of using this function would look like this.

{% highlight go %}
package main

func main() {
    server := NewHttpServer("0.0.0.0", 8080)
    server.ListenAndServe()
}
{% endhighlight %}

Happy coding!