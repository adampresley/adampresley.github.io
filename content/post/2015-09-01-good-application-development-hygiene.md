---
author: "Adam Presley"
comments: true
date: 2015-09-01T00:00:00Z
tags: ["development"]
title: "Good Application Development Hygiene"
slug: "good-application-development-hygiene"
---

![Woman Brushing Teeth](/assets/adampresley/images/posts/woman-brushing-her-teeth.png)

The other day I was reading a [cool blog post](http://www.morethanseven.net/2015/08/21/operating-unikernel-challenges/) where the author was musing on what he considered would be challenges for the up and coming trend that is unikernels. I was enjoying the read when a single phrase from the post jumped out at me and caught my attention.

> "Part of this is good application development hygiene..."

The author uttered the phrase almost in passing while talking about the challenges of debugging an application written as a unikernel versus the traditional OS + Network + Application stack. As I finished I couldn't help but reach for my handy-dandy Sublime text editor and jot down a question. *What does it mean to practice **good application development hygiene**?*

<!-- excerpt -->

### What It Means To Me
Here in this blog post I will describe what **Good Application Development Hygiene** means to me. I suspect that the definition can vary from person to person, and I would even venture to say that my own perspective has, and will continue to change over time. For now I will simply describe some high-level ideas that I feel represent good application development hygiene.

Much like brushing your teeth, exercising, and having regular checkups with your doctor to keep your body healthy, software applications require you to perform maintenance and check in on it from time to time to keep it running smoothly. A healthy software application keeps your customers happy, your engineering team sane, and can potentially reduce the impact of costly maintenance and bug fixes when errors and crashes *do* happen.

A lot of the things that make software applications run continuously and without interruption isn't sexy, and can be very mundane. It can often be a hard sell to convince your sales and product management teams that you need to upgrade a server or install and test a big security patch, because let's face it, you can't put a subscription fee on that and sell it. Plugging up a cross-site scripting vulnerability or refactoring that 5,000 line piece of code do not make good bullet points on your marketing material, but ultimately by following best practices and being aware of good application development hygiene can reduce the cost of building future features, bug fixes, and your customers will have a solid, reliable product.

### Some Specific Strategies
Below I will devle into a little more detail on some things that software companies and individual developers alike can do to practice good application development hygiene.

#### Logging
Logging is the act of recording information about some event at the time that it occurs. As an example, when a user logs into your application with their user name and password you may record who the user was, what their IP address is, complete with a date and time stamp. Perhaps your customers wish to report on when one of their employees alter the price of an inventory item in the application, or maybe you need to keep up with all requests for information for government regulation purposes.

In any case there are plenty of reasons to implement a solid logging strategy. With appropriate logging you have better visibility into what events transpired in your application. If a user reports a crash you now have some context of what led up to the event. Did an employee claim they logged in and ran that report? With a good logging strategy you could have proof to the contrary. And all of that is at the application level. This does not include logging at the infrastructure level, such as web server, firewall, and other very useful information sources.

#### Stats
This particular point might be contentious. I suggest to you that tracking some level of statistics on how your application is performing can be very useful. How much to record I leave as an exercise to the reader. However I have found value in recording such information as web requests and their timings, hits and misses in an application cache, memory consumption trends, and more. At some point in the lifecycle of an application you will need to know more details about what *really* happens when the code is running. How much RAM is consumed when I run that big report? Are garbage collection pauses really causing us to violate our request time metric?

If you start to consider the real possiblity that someone other that *you* is going to run your application in an environment *other* than your local machine, you'll realize that you may need to have a way to measure how the application is performing. There are several reasons why you would need to know this. You may have availability metrics specified in your contract with a particlar customer. Maybe you are getting support emails about slow running dashboard graphs. All of these require you to have the ability to peek under the covers at the execution environment, and it is better to have some of those hooks in place early in an application's life. Adding them later can sometimes be difficult and costly.

#### Testing
This one should be a no-brainer. Testing is the act of proving that your application does what it says on the label. If you implement a new feature, or fix a bug, or any other activity that *changes* the software, you should be testing. In fact you should be testing on multiple levels. The art and science of software testing has evolved quite a bit over the last ten years. These days if you don't have some strategy for running more tests through automation than you run manually, I'd argue you have a lot of room for improvement.

To get the biggest bang for your buck as a software company ensure you are investing maximum energy in unit testing. Fundamentally all software boils down to functions working on data. Functions are usually the smallest unit of work in a software application and are the foundation of what makes it all tick. As such it makes a lot of sense to ensure that these functions perform their jobs well. Unit testing does exactly that.

Once you have sufficent unit testing habits take it a step further. Automate their execution. Set up your development and testing environments so that unit tests are run regularly and ensure they report failure **immediately** to your engineering team. The sooner developers have that feedback the sooner in the process they can fix the problem. The sooner the problem is fixed the less money you as a business spend. I don't know about you, but I like money.

Perhaps less sexy than automated unit tests and continuous integration is the regression test. Regression testing is testing to ensure that a change to the software doesn't cause something else to revert back to a broken state. As software grows more complex and large this is an area of practice that requires a great deal of disclipline and planning. However a good regression testing strategy can save you tons of hours of heartache and embarrasment.

Get in the habit. Test.

#### The Payment Plan
Over time software starts to show its age. As a software developer, product manager, engineering manager, quality specialist, or any of the other titles that go into crafting software, you will have to make choices that affect what you build, and how you build it. After a while you will hear someone utter something about the software "having technical debt". Technical debt is the consequence of design, architectural, and coding decisions made over the life of an application. ***All software incurs technical debt.***

Technical debt isn't always bad, and it can be built up over time for various reasons. Sometimes trade-offs must be made in order to make the sale, or perhaps the business couldn't afford that awesome component that would have made the application epic. Other times technical decisions are made without the full benefit of the whole business problem, which can lead to the software not fullfilling a customer's needs long term. Technical debt can also build up by virtue of the ever-changing technical landscape. Technology changes, and sometimes changing your software along with the times, can be cost prohibitive.

Regardless of how the debt is accrued one thing is certain. *You **MUST** commit to a payment plan!* Much like financial debt, technical debt grows and compounds over time. If you choose to ignore or put off payment on some of that debt, at some point, something will break, and the cost of repair may be more than you can swallow. As a strategy and matter of good habit I cannot stress enough the importance of treating technical debt items as first class citizens in your product bug backlog. These are things that must be worked alongside your sexy new features and super-cool disruptive mobile app technology.

#### Summary
A lot of what I mentioned above could be considered common sense to some, or enlightening to others. I've often observed that what I might consider *good application development hygiene* isn't always known or considered. As such I feel it is a good idea, as a developer working alone at night, or on a team of one hundred, to take some time to define what you think is good hygiene. Write down a strategy that will help you build excellent software and please customers. Then stick to it, and be disciplined. Your future software will thank you.
