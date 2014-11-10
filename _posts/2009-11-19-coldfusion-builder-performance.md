---
layout: post
title: ColdFusion Builder Performance
date: 2009-11-19 03:22:00
author: Adam Presley
status: Published
tags: development tools eclipse
slug: coldfusion-builder-performance
---

I don't know about you guys, but I run into horrible performance issues
as I use ColdFusion Builder, or most other Eclipse-based IDEs, as I use
them more over time, add more projects, etc... After researching I've
got a set of settings now that help a bit. These settings make garbage
collection more aggressive, which is proving decent.   
  
To modify your CF Builder settings you need to open your CFBuilder.ini
file. In Windows this is located at **C:\Program Files\Adobe\Adobe
ColdFusion Builder** by default. If you open that up you can setup the
parameters the Java Virtual Machine will run with for Eclipse. Here is
what I use. Maybe someone will find it useful.  
  
	:::ini
	-startup
	plugins\org.eclipse.equinox.launcher_1.0.101.R34x_v20081125.jar
	--launcher.library
	plugins\org.eclipse.equinox.launcher.win32.win32.x86_1.0.101.R34x_v20080731
	-vmargs
	-Dosgi.requiredJavaVersion=1.6
	-Xms128m
	-Xmx384m
	-Xss2m
	-XX:PermSize=128m
	-XX:MaxPermSize=128m
	-XX:MaxGCPauseMillis=10
	-XX:MaxHeapFreeRatio=70
	-XX:+UseConcMarkSweepGC
	-XX:+CMSIncrementalMode
	-XX:+CMSIncrementalPacing
	-XX:CompileThreshold=5
	-Dcom.sun.management.jmxremote
