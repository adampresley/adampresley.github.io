---
layout: post
title: PayPal Frustrations
date: 2010-02-05 10:48:00
author: Adam Presley
status: Published
tags: development
slug: paypal-frustrations
---
Today I've been working on the final bit of a PHP contract for a client
where I am converting an older ASP site to PHP. This phase involves
moving over the PayPal code. The API is pretty simple for PayPal
actually, and involves sending a POST to a specific API with a specific
format. So basically a RESTful service.  
  
I'm not one to just slap up the code and test *live*, however, and
wanted to do some testing first to make sure the work flow for this part
of the application is correct. So I'm going over the documentation,
admittedly in a hurry, and see that I need to use a sandbox URL. Ok, I
try that, but get odd **internal errors**. I find that odd, but
start pouring over a Google Search in an attempt to see who else has
seen this and already solved said issue.  
  
After quite a lot of getting nowhere, I finally see something that tells
me that you can't just use the sandbox URL with your normal PayPal
credentials. You have to setup a separate sandbox account. Ooookkaaay...
Fine.  
  
I set out to do this task, and I create an account. I find this easy
enough, and I also find setting up a "cookie cutter" pre-made accout for
a buyer easy enough. But when I go to test that using the Website Direct
Pay Pro thingie, it complains that my sandbox is not setup correctly. So
I start to investigate this. Turns out the pre-baked accounts do not use
this, and you have to set one up manually.  
  
**FINE**. I start that process. I get past page one where I put in
my email and password, and accept the terms, and then fill out all of
page two with who my "business" is. When I submit this, however, I get a
cryptic error. Come to find out my password from the **PREVIOUS**
step is not valid, and it didn't bother to tell me till I submitted step
2!!  
  
By this point I'm ready to start sniping imaginary targets in thin air
with my imaginary finger-rifle. But I move forward. I get the account
setup. All I have to do now is verify the information in the email they
sent me. Oh do I??? What email!! I asked for it 4 times, and never got
it today. Yes, I checked Junk mail.   
  
I'd like to go on the record and wonder how in the world PayPal has
become one of the most popular solutions in the world if setting up a
dev environment is this ... icky!! GAH!
