---
author: "Adam Presley"
date: 2014-05-30T03:37:00Z
tags: ["development", "golang"]
title: "Parsing Woes in MailSlurper"
slug: "parsing-woes-in-mailslurper"
---

Tonight as I was working a bug in MailSlurper I quickly came to the realization that *my parsing routine sucks*. The current version is passable for simple text emails or basic HTML emails. But the moment you add attachments or inline images it all goes downhill. The further I dove into this problem the more I also realized that RFC 2822, or *Internet Message Format* and MIME are also a little dated and weird.

<!-- excerpt -->

Basically a mail is broken down into two major parts. The first part is headers. Headers are key/value pairs separated by a colon, each set separated by a carriage return and line feed (\r\n for you nerds out there). This describes what to expect, such as the subject of the message, the sender, recipients, and more. After that another set of **\r\n** indicates we are ready for the second half of the mail, which is called the **body**. This contains the content of your email message. It also houses an HTML version of your mail if you used fancy things like links and italics, and it will also have Base64 encoded data for each attachment in your email.

To separate the mail message from the attachments the MIME specification defines what is called a **boundary**. This is an identifier in a specific format that comes before each attachment or section of the body. In learning all this though I started to see some complexity creep in. When the email is multipart, or contain attachments or pretty body content (HTML), the boundary gets slapped in on the end of the *Content-Disposition* header. So if *Content-Disposition* had a value already, it now has a semicolon, then the boundary marker definition.

Attachments commit the same evil in their version of the *Content-Disposition* header by having the file name follow after a semicolon. The is unfortunate because it muddies up my header parsing piece. I now have to be on the lookout for specific headers having additional information, which changes how I need to parse the body. If there is no boundary marker then I don't have any attachments or HTML to pull out. If I do, then parsing headers for attachments I have to look at that SAME header name of *Content-Disposition* but look for a different key for the file name. Ick.

There isn't really a point to this post except for me to rant and organize my thoughts a bit. This won't be hard to do, but it will take a bit of reogranizing my parsing code.