---
layout: post
title: Response To Adam Cameron's Code Review
date: 2014-12-18 08:30:00
author: Adam Presley
tags: development golang
comments: true
---
A few weeks ago I submitted a [response]({% post_url 2014-11-10-a-go-weekend-puzzler %}) to a code puzzle on [Adam Cameron's blog](http://blog.adamcameron.me) that I wrote using Go. He has (finally) submitted a review! First let me say that it is about time Mr. Cameron! I have waited with bated breath! That review can be read [here](http://blog.adamcameron.me/2014/12/weekend-quiz-adam-presleys-answer-go.html). In this post I will comment on his commentary.

<!-- excerpt -->

#### YAGNI
Mr. Cameron asserts in his post that I had too much "scope creep", went a "bit overboard", and that there was simply no need to go to such lengths for a simple puzzle. I will confess that I certainly took this to the next level, and certainly without prompt. In fact, I violated the YAGNI principle. For those who are unfamiliar with YAGNI it stands for *You Ain't Gonna Need It*. This principle pretty much states that if you aren't going to need it, don't build it. There was no indication in the specifications that I would need parallel processing, or that I would have large data sets to work with. I didn't violate the specifications per se, but I certainly went far above what was really **needed**.

So why would I do this? For two reasons really.

1. For the fun of it! I wanted to try some fun stuff in my solution
2. To ensure I aced the 3rd criteria that Mr. Cameron indicated would help one win. Here is the line from his post:

    > Answers that I find more technically interesting are more likely to win than answers that are pedestrian
    > or obvious. Scalpels rather than sledgehammers.

I certainly feel my solution is technically interesting, and excels at that criteria. Of course I do empathize with Mr. Cameron in the fact that *grokking* that code would take some effort.

#### The errors
Adam did find a couple of issues that were great catches! The first error happens when you pass no input array, and you get an error. That error is actually thrown when attempting to convert values from the console to integers, so I am actually throwing that error. This means I am catching it. However a quick check for a blank input and returning more user friendly messaging would go a long way, I agree.

The second catch involves setting a threshold value that is less than any of the individual array items. This is a good find, and one I should have thought to try in my unit tests.

#### How To Test
Mr. Cameron mentions he had some difficulty running the tests. Assuming the code compiles there are two steps necessary to run the tests. The first is to install [GoConvey](https://github.com/smartystreets/goconvey) and the second is to execute the tests.

{% highlight bash %}
$ go get https://github.com/smartystreets/goconvey
$ go test ./... -v
{% endhighlight %}

Once you execute this you should see something like this.

![Screenshot 1](/assets/adampresley/images/posts/adamCameronPuzzle-tests-screenshot1.png)

If you want to see the pretty graphic interface, do the following on the command line after installing GoConvey.

{% highlight bash %}
$ $GOPATH/bin/goconvey
{% endhighlight %}

![Screenshot 2](/assets/adampresley/images/posts/adamCameronPuzzle-tests-screenshot2.png)

#### Follow-up
So at Adam's request I will do a simpler version of this code.

{% highlight go %}
package main

import (
    "flag"
    "fmt"
    "os"
    "strconv"
    "strings"
)

type SubArrayResult struct {
    Sequence []int
    Total    int
}

var input = flag.String("input", "", "Comma delimited list of integer numbers")
var threshold = flag.Int("threshold", 10, "Integer number for array slice to not exceed")

func main() {
    flag.Parse()

    /*
     * Array of integers to determine longest sequence from. These are a string
     * list on the command line that are converted to integers.
     */
    inputArray := make([]int, 0)

    if *input == "" {
        fmt.Println("Please provide a valid input of comma delimited integers")
        os.Exit(1)
    }

    /*
     * Get the input from the console args, and split them up
     * into integers in an array
     */
    for _, value := range strings.Split(*input, ",") {
        i, err := strconv.Atoi(value)
        if err != nil {
            fmt.Println("ERROR - Unable to convert", value, "to integer")
            os.Exit(1)
        }

        inputArray = append(inputArray, i)
    }

    /*
     * Loop, slice, sum
     */
    sequences := make([]SubArrayResult, 0)

    for index, _ := range inputArray {
        sequence := make([]int, 0)
        total := 0

        inputSlice := inputArray[index:]

        for _, value := range inputSlice {
            if (total + value) <= *threshold {
                total += value
                sequence = append(sequence, value)
            } else {
                break
            }
        }

        if len(sequence) > 0 {
            result := SubArrayResult{
                sequence,
                total,
            }

            sequences = append(sequences, result)
        }
    }

    /*
     * Who wins?
     */
    longestIndex := 0
    longestLength := 0

    for index, item := range sequences {
        if len(item.Sequence) > longestLength {
            longestLength = len(item.Sequence)
            longestIndex = index
        }

        if len(item.Sequence) == longestLength && item.Total > sequences[longestIndex].Total {
            longestLength = len(item.Sequence)
            longestIndex = index
        }
    }

    if len(sequences) > 0 {
        winningSequence := sequences[longestIndex]

        fmt.Println("")
        fmt.Println("Sequence", winningSequence.Sequence, "wins with a length of", len(winningSequence.Sequence), " for a total of", winningSequence.Total)
    } else {
        fmt.Println("No winners :(")
    }
}
{% endhighlight %}

Cheers, and happy coding!!

