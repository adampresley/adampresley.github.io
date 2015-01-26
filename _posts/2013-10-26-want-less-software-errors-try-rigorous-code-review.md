---
layout: post
title: Want Less Software Errors? Try Rigorous Code Review
date: 2013-10-26 04:56:00
author: Adam Presley
status: Published
tags: development
slug: want-less-software-errors-try-rigorous-code-review
---
<img src="http://s3.amazonaws.com/www.adampresley.com/posts/python-code-angled.png" class="pull-left" style="margin-right: 10px; margin-bottom: 10px;" />
Anyone who has been in the software development field for more than a few
years, contributing to projects in a team environment, knows that one key
stategy for reducing the number of errors in a software application
is to implement some type of review/inspection process. Be it an open source
project with contributors across the globe or co-located team members
in an enterprise organization the more critical eyes you have on a given
piece of code the better the outcome.

<!-- excerpt -->

Let us start with a definition of code review. Wikipedia defines code review as
a "...systematic examination (often know as peer review) of computer source code.
It is intended to find and fix mistakes overlooked in the initial development
phase, improving both the overall quality of software and the developers' skills."
(<http://en.wikipedia.org/wiki/Code_review>). Essentially code review is a process
of having peers not always involved in the implementation of code examine,
critique, understand and decompose a given piece of source code, with the intent
of finding errors that may have been missed. This process can also be used
to examine other possible approaches to the solution to ensure the problem
has been properly examined, understood, and addressed.

**Rigorous** code review takes the concept of simply reviewing code
and raises it to the next level. It declares that individuals or teams
reviewing the code take the time to understand each line of code. This
involves reviewing the impact, the purpose, the circumstances driving
the decisions that led to the solution in review, and so on. During this
process each line of code, each decision should be questioned.

I personally have reviewed a good deal of code over time, but to truely
comb through it is something I am *not* practiced at. Here are a few
bullet points that highlight my experience so far that you might find useful.

* Analyze the following in methods:
    * What are its inputs?
    * What are the outputs?
    * What exceptions can be thrown?
    * What are the dependencies?
        * Methods
        * Non-local variables
        * Components
        * Web services
        * Database
    * Did inputs change?
    * Did outputs change?
    * What are the side-effects of this method?
* Question every unit test
    * Is there enough coverage?
    * Are the current test methods logical?

The list above is certainly not comprehensive but I hope it illustrates
the fact that a real code review digs deep. If I don't understand what
some method does, or what a particlar variable or class is for, dig in
and find out. When in doubt ask these questions of the people who
wrote the code.

So why perform rigorous code review? Why put so much effort into learning
and digging into what someone else has already spent time and effort on?
In **Facts and Fallacies of Software Engineering** by *Robert L. Glass*
(Addison-Wesley Professional) the author contends that "Rigorous inspections
can remove up to 90 percent of errors from a software product before the
first test case is run." (Also see Wiegers, Karl E. 2002 *Peer Reviews
in Software - A Practical Guide.* Boston: Addison-Wesley). Mr. Glass goes on
to talk more about how reviews are not a magic bullet and certainly not
a replacement for other testing methods, but this does indicate
a strong case for subjecting code to rigorous review prior to testing
and release.

I look forward to learning more and becomming a better reviewer of code
and if the research studies of how review affects quality are anything
I look forward to releasing higher quality code. Happy reviewing!