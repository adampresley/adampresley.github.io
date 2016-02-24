---
author: "Adam Presley"
comments: true
date: 2014-12-15T22:31:00Z
tags: ["development", "golang"]
title: "Handling CTRL+C In Go Command Line Applications"
slug: "handling-ctrl-c-in-go-command-line-applications"
---

Command line and server-style application often allow you to press a key combination, such as **CTRL+C** to gracefully exit. In Go this is a pretty easy task to accomplish, and in this entry I will show you how you can add this ability in your own applications.

<!-- excerpt -->

First off, **CTRL+C** is referred to, especially in the \*nix world, a **signal**. The proper name for the signal sent to your application when you press **CTRL+C** is **SIGINT**. This is short for an *interrupt signal*. The Go standard library has support for receiving and handling signals in your application.

To handle **SIGINT** in your application you need to perform three steps

1. Setup a channel to receive a notification when the signal occurs
2. Setup the notification to be sent to that channel
3. Have a goroutine that loops and waits until something is received on that channel

For this example let's look at how we can listen for **SIGINT** and simply exit the application when that signal is received.

{{< highlight go >}}
import (
    "os"
    "os/signal"
)

func main() {
    // Setup a channel to receive a signal
    done := make(chan os.Signal, 1)

    // Notify this channel when a SIGINT is received
    signal.Notify(done, os.Interrupt)

    // Fire off a goroutine to loop until that channel receives a signal.
    // When a signal is received simply exit the program
    go func() {
        for _ = range done {
            os.Exit(0)
        }
    }()
}
{{< / highlight >}}

The tools to handle signals are part of the standard Go library. In lines 1 - 4 we see the imports for this lie in the *os* and *os/signal* packages. After importing the necessary libraries we then setup a channel which has room for a single item of type **os.Signal**. Then we call **Notify()** to write to the channel when a **SIGINT** (os.Interrupt) is received.

Finally we spawn a goroutine that starts a loop that will block execution until that channel receives something. When it does we are simply exiting the application by calling **os.Exit()**. Now let's look at a version of this that uses a little package I put together that you can find on [Github](https://github.com/adampresley/sigint).

{{< highlight go >}}
import (
    "os"

    "github.com/adampresley/sigint"
)

func main() {
    sigint.ListenForSIGINT(func() {
        os.Exit(0)
    })
}
{{< / highlight >}}

Notice that the code is similar, but this package allows you to call a function, passing in another function that will be executed when a **SIGINT** is fired off to your application.

Cheers, and happy coding!