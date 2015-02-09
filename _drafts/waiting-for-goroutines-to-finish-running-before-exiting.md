---
layout: post
title: Waiting For Goroutines to Finish Running Before Exiting
date: 2015-02-15 22:00
author: Adam Presley
tags: development golang
comments: true
---
![Swirly Lights](/assets/adampresley/images/posts/steelwool.jpg)
*Image copyright [Faizal Sugi](http://pixabay.com/en/steelwool-dark-firespin-spiral-art-458840/)*

Goroutines are primitives in Google's Go language that allow a developer to easily write concurrent applications to make use of modern multi-core CPUs. Because these are baked into the language and super easy to use it becomes simple to make quick use of them in your applications. As you start to write more sophisticated server applications you realize you need a way to gracefully exit your application. This means that you may not want your application to exit before all your Goroutines are done executing. In this entry I'll talk about one way to handle this with a ```sync.WaitGroup``` and channels.

<!-- excerpt -->

### Simple Example

Let's start with a simple example. Here we will create a simple application which creates a Goroutine that loops continuously and performs some imaginary function over and over. We will also have a SIGINT/SIGTERM listener so that we can exit our application if CTRL+C is pressed.

{% highlight go %}
package main

import (
   "os"
   "os/signal"
   "syscall"
)

func main() {
   /*
    * When SIGINT or SIGTERM is caught write to the quitChannel
    */
   quitChannel := make(chan os.Signal)
   signal.Notify(quitChannel, syscall.SIGINT, syscall.SIGTERM)

   /*
    * Create a goroutine that does imaginary work
    */
   go func() {
      for {
         // Do some hard work here!
      }
   }()

   /*
    * Wait until we get the quit message
    */
   <-quitChannel
}
{% endhighlight %}

#### What's wrong with this?

So what is wrong with this? This biggest issue is that when the application shuts down you may be doing something important in this Goroutine. For example you could be writing queued data to a database. If this Goroutine gets interrupted you could be losing data! So how do we stop this? We'll need two tools: a channel and a ```sync.WaitGroup```.

### A Shutdown Channel

First we need a way to signal to Goroutines that shutdown is imminent, and that they need to exit their loops. A channel is the perfect way to do this. First let's create that channel.

{% highlight go %}
shutdownChannel := make(chan bool)
{% endhighlight %}

That was easy. Next we need to pass it around to any Goroutine that intends to run into infinity.

{% highlight go %}
go func(shutdownChannel chan bool) {
   for {
      // Do some hard work here!
   }
}(shutdownChannel)
{% endhighlight %}

Finally let's write the code that will exit the Goroutine if there is a message on the shutdown channel. This bit of code uses the ```select``` statement to listen on channels for messages. In our case if the shutdown channel gets a message we want to exit the Goroutine. For more information on the ```select``` statement see the [GoLang Tutorials](http://golangtutorials.blogspot.com/2011/06/channels-in-go-range-and-select.html) website.

{% highlight go %}
go func(shutdownChannel chan bool) {
   for {
      /*
       * Listen on channels for message.
       */
      select {
      case _ = <-shutdownChannel:
         return

      default:
      }

      // Do some hard work here!
   }
}(shutdownChannel)
{% endhighlight %}

### Waiting For Them to Finish

So now we have a way to tell Goroutines to wrap it up, but we still need a way to wait for them to finish in the event that they are still working on stuff. As it turns out the Go standard library has something for this in the ```sync``` package called a ```WaitGroup```. The idea is easy. You add one (1) every time you start up a Goroutine or some process that you need to wait on. You do this by calling ```WaitGroup.Add(1)```. When that process or Goroutine is done you call ```WaitGroup.Done()``` to decrement the counter. Then when you are ready to exit, but want to ensure that all Goroutines and processes are complete first, you can call ```WaitGroup.Wait()``` to block execution until the wait group's counter gets to zero (0). Let's take a look at that.

{% highlight go %}
waitGroup := &sync.WaitGroup{}

/*
 * Increment the wait group because we are starting up a Goroutine
 */

waitGroup.Add(1)

go func(shutdownChannel chan bool, waitGroup *sync.WaitGroup) {
   defer waitGroup.Done()

   for {
      /*
       * Listen on channels for message.
       */
      select {
      case _ = <-shutdownChannel:
         return

      default:
      }

      // Do some hard work here!
   }
}(shutdownChannel, waitGroup)
{% endhighlight %}

Notice how we add one to the wait group before starting our Goroutine process. Then notice that we defer a call to ```WaitGroup.Done()``` so that when the shutdown channel returns out of the loop the wait group counter will be decremented.

Now all we need to do is put it all together. Once the SIGINT or SIGTERM signal is received and the application starts to quit we'll want to wait for the wait group count to get to zero, *then* we can exit. Here is the final source code.

{% highlight go %}
package main

import (
   "os"
   "os/signal"
   "sync"
   "syscall"
)

func main() {
   /*
    * When SIGINT or SIGTERM is caught write to the quitChannel
    */
   quitChannel := make(chan os.Signal)
   signal.Notify(quitChannel, syscall.SIGINT, syscall.SIGTERM)

   shutdownChannel := make(chan bool)
   waitGroup := &sync.WaitGroup{}

   waitGroup.Add(1)

   /*
    * Create a goroutine that does imaginary work
    */
   go func(shutdownChannel chan bool, waitGroup *sync.WaitGroup) {
      defer waitGroup.Done()

      /*
       * Listen on channels for message.
       */
      select {
      case _ = <-shutdownChannel:
         return

      default:
      }

      for {
         // Do some hard work here!
      }
   }(shutdownChannel, waitGroup)

   /*
    * Wait until we get the quit message
    */
   <-quitChannel

   /*
    * Block until wait group counter gets to zero
    */
   waitGroup.Wait()
}
{% endhighlight %}

### Conclusion

Goroutines and channels offer very powerful mechanisms for concurrent programming. These patterns still require a bit of discipline to write a safe, stable application, however. Hopefully this helps demonstrate a simple example of how one might ensure that concurrent operations are complete prior to exiting your application. Cheers, and happy coding!
