---
layout: post
title: Consuming Webservices with Self-signed Certificates in C#
date: 2011-06-23 16:26:00
author: Adam Presley
status: Published
tags: development c#
slug: consuming-webservices-with-self-signed-certificates-in-c
---
Ran into a little issue today while writing a C# app to do unit testing
of a ColdFusion-based webservice API. The ColdFusion service is running
on a development server that has a self-signed certificate installed in
IIS for HTTPS testing. When I finished up my C# application and
attempted to connect and execute one of the services I got an error:
"The underlying connection was closed: Could not establish trust
relationship for the SSL/TLS secure channel."

Fortunately a little Googling showed the way to those who have solved
this issue before me. This solution is pretty simple, though Visual
Studio complains that this method is deprecated. This doesn't bother me,
because it works. :)

First create a new class in your project. Let's call the file
**CertValidation.cs**. In this file we'll put in some code that
implements a specific interface designed to handle certificate
validation. Except when the validation method is called we won't bother
to validate, but instead will just tell the app that all is well!

	:::c#
	using System;
	using System.Net;
	using System.Security.Cryptography.X509Certificates;

	namespace ourCoolApp {
	 public class CertValidation : ICertificatePolicy {
	  public Boolean CheckValidationResult(ServicePoint srvPoint, X509Certificate cert, WebRequest request, Int32 problem) {
	   // Return true to indicate the cert is ALWAYS valid. Hacky-tastic!
	   return true;
	  }
	 }
	}

Once you have that saved you will need to tell your application to use
this for certificate validation. I did exactly this in the Load event in
my main form like this.

	:::c#
	ServicePointManager.CertificatePolicy = new CertValidation();

Make sure to import **System.Security.Cryptography.X509Certificates** in
your form code, or that will fail. Now when you run your application,
the *CertValidation* class will tell your application that the cert is
cool!

Happy coding!
