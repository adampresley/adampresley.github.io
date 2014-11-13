---
layout: post
title: Beep on the Enter Key
date: 2007-08-27 12:20:00
author: Adam Presley
status: Published
tags: development c# .net
slug: beep-on-the-enter-key
---
One of the applications that I wrote and maintain here for use at my job
is a C# Windows application. A couple of revisions ago I set it up so
that the user could press the **Enter** key to submit the login instead
of having to click the actual login button. For the longest time now
this has produced a ***beep*** sound. ANNOYING!  
  
I finally took the time to address this. On the text boxes I was
catching the *KeyUp* event and checking for the **Enter** key. Come to
find out that to properly make sure that the **WHOLE** event gets
handled I needed to use the *KeyPress* event, and set the ***Handled***
property to *true* so that it would not flow back down to the key event
handler on the form. Now, no more beep!
