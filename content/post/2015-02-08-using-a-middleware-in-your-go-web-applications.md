---
author: "Adam Presley"
comments: true
date: 2015-02-08T00:00:00Z
tags: ["development", "golang"]
title: "Using A Middleware in Your Go Web Applications"
slug: "using-a-middleware-in-your-go-web-applications"
---

![Bikes Lined Up](/assets/adampresley/images/posts/62H.jpg)
*Image copyright [Ryan McGuire](http://www.gratisography.com/)*

Middleware in Go HTTP applications is a way to chain HTTP handlers in a sequence to alter the request and response chain prior to your primary handler getting it. This is useful for activities such as logging, input sanitization, handling authorization and security, and more. This concept is supported in the Go standard library with functions like [http.StripPrefix](http://golang.org/pkg/net/http/#StripPrefix) and [http.TimeoutHandler](http://golang.org/pkg/net/http/#TimeoutHandler). Each of the previously mentioned functions take a reference to a handler and return a new handler, wrapping the original with its own logic and processing of the request and response chain. In this post I will show how you can use a middleware called [Alice](https://github.com/justinas/alice) to perform this task.

<!-- excerpt -->

### First Middleware Function

First let's take a look at what a middleware function looks like. Alice is a particularly beautiful solution because of how simple it is. It doesn't try to get fancy, or do any weird reflection nonsense. Alice simply chains HTTP handlers together to alter request and response. As such an Alice middleware function is simple. It takes one argument of type ```http.Handler``` and returns an ```http.Handler```. So for the sake of this example let's assume we wish to display some log information to the console for each request. Perhaps the HTTP method and path information. And maybe we can also display how long the request took to run. Here's what such a middleware method would look like.

{{< highlight go >}}
func Logger(h http.Handler) http.Handler {
   return http.HandlerFunc(func(writer http.ResponseWriter, request *http.Request) {
      startTime := time.Now()
      h.ServeHTTP(writer, request)
      log.Printf("%s - %s (%v)\n", request.Method, request.URL.Path, time.Since(startTime))
   })
}
{{< / highlight >}}

The above function takes in an HTTP handler and grabs the current timestamp. It then executes the passed in handler. Finally it logs to the console the HTTP method (VERB), the path information, and how much time elapsed from the start timestamp. That was easy!

### Chaining Them Together

Now that we have a middleware wrapper let's chain them together. In a [previous post]({% post_url 2014-11-18-returning-an-http-server-from-a-function-in-go %}) I demonstrated returning an HTTP listener from a function. I am going to build on that by taking the result from that function and wrapping Alice around it, giving your handlers a middleware wrapper. This is the code we setup previously to get an HTTP listener.

{{< highlight go >}}
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
{{< / highlight >}}

Let's modify the **NewHttpServer()** function a bit. Here is where we will take our HTTP router and add Alice. Alice is easy to setup. To start you call **New()** passing in each handler you want to execute, *in the order that you want them to execute*. That returns a structure off which you will call **Then()**, which tells Alice the final handler that must execute, which needs to be our router. This sample assumes the logger middleware function we setup previously is in a directory named **middleware**. Here's how it comes together.

{{< highlight go >}}
func NewHttpServer(address string, port int) *http.Server {
   router := setupHttpRouter()

   server := alice.New(middleware.Logger).Then(router)

   listener := &http.Server{
      Addr:    fmt.Sprintf("%s:%d", address, port),
      Handler: server,
   }

   return listener
}
{{< / highlight >}}

### Screenshot

![Example logger middleware](/assets/adampresley/images/posts/example-logger-middleware.png)

### Code Listing

Here is the code in full.

#### middleare/Logger.go

{{< highlight go >}}
package middleware

import (
   "log"
   "net/http"
   "time"
)

func Logger(h http.Handler) http.Handler {
   return http.HandlerFunc(func(writer http.ResponseWriter, request *http.Request) {
      startTime := time.Now()
      h.ServeHTTP(writer, request)
      log.Printf("%s - %s (%v)\n", request.Method, request.URL.Path, time.Since(startTime))
   })
}
{{< / highlight >}}

#### listener/HttpListener.go

{{< highlight go >}}
import (
   "fmt"
   "net/http"

   "github.com/adampresley/myproject/controllers/versionController"

   "github.com/gorilla/mux"
   "github.com/justinas/alice"
)

func setupHttpRouter() http.Handler {
   router := mux.NewRouter()
   router.HandleFunc("/", versionController.GetVersion).Methods("GET", "OPTIONS")

   return router
}

func NewHttpServer(address string, port int) *http.Server {
   router := setupHttpRouter()

   server := alice.New(middleware.Logger).Then(router)

   listener := &http.Server{
      Addr:    fmt.Sprintf("%s:%d", address, port),
      Handler: server,
   }

   return listener
}
{{< / highlight >}}

Cheers, and happy coding!
