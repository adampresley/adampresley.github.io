---
layout: post
title: Grails - Problem Transforming Call Sites
date: 2012-08-29 19:50:00
author: Adam Presley
status: Published
tags: development groovy grails
slug: grails-problem-transforming-call-sites
---

While working on a project in Grails 2.1.0 (on Windows 7 64-bit) I ran
into an interesting error.

    :::txt
    Aug 29, 2012 7:17:21 PM com.springsource.loaded.agent.SpringLoadedPreProcessor preProcess
    SEVERE: Unexpected problem transforming call sites
    java.lang.IllegalStateException: Unexpected problem processing bytes for class
            at com.springsource.loaded.ConstantPoolChecker2.readConstantPool(ConstantPoolChecker2.java:182)
            at com.springsource.loaded.ConstantPoolChecker2.(ConstantPoolChecker2.java:114)
            at com.springsource.loaded.ConstantPoolChecker2.getReferences(ConstantPoolChecker2.java:88)
            at com.springsource.loaded.MethodInvokerRewriter.rewrite(MethodInvokerRewriter.java:248)
            at com.springsource.loaded.MethodInvokerRewriter.rewriteUsingCache(MethodInvokerRewriter.java:141)
            at com.springsource.loaded.TypeRegistry.methodCallRewriteUseCacheIfAvailable(TypeRegistry.java:775)
            at com.springsource.loaded.agent.SpringLoadedPreProcessor.preProcess(SpringLoadedPreProcessor.java:251)
            at com.springsource.loaded.agent.ClassPreProcessorAgentAdapter.transform(ClassPreProcessorAgentAdapter.java:89)
            at sun.instrument.TransformerManager.transform(TransformerManager.java:188)
            at sun.instrument.InstrumentationImpl.transform(InstrumentationImpl.java:424)
            at java.lang.ClassLoader.defineClass1(Native Method)...

I had done some searching and apparently a *similar* error had been
addressed previously. The part about **Unexpected problem transforming
call sites** was the same, but the actual exception,
**java.lang.IllegalStateException** differed. I'm not sure what the
*actual* fix is, but I went to *{home folder}/AppData* and deleted the
**.grails** folder. This seemed to address the problem. Hope this helps
anybody who might run into the same problem.
