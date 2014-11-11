---
layout: post
title: Configuring Illudium PU-36 Code Generator on Ubuntu Linux
date: 2009-07-05 11:30:00
author: Adam Presley
status: Published
tags: development coldfusion
slug: configuring-illudium-pu-36-code-generator-on-ubuntu-linux
---

In an effort to get up to speed on ColdBox, an MVC application framework
for ColdFusion, I wanted to make use of the very cool code generator
called Illudium PU-36. Clearly the author is a Looney Toons fan. I've
spent the last few days getting my old Dell Inspiron 9300 running with
Ubuntu Linux 9.04. Shout-out to those folks for an **AWESOME**Linux
distribution. So I've got LAMPP setup with Apache, and ColdFusion 8
developer edition all ready to rock.  
  
I proceed to download Illudium PU-36 and extract it to a folder. First
thing, you must name your folder *cfcgenerator*. I then moved into
Apache config files and setup the virtual host entry, as well as an
entry in **/etc/hosts** so I can reference the site as
**http://cfcgenerator.dev**.  
  
{% highlight apache %}
<virtualhost *:80>
	ServerAdmin webmaster@codegenerator.dev
	ServerAlias cfcgenerator.dev
	DocumentRoot "/home/psykoprogrammer/code/coldfusion/cfcgenerator"
	ServerName cfcgenerator.dev

	<directory "/home/psykoprogrammer/code/coldfusion/cfcgenerator">
	Options Indexes FollowSymLinks
	AllowOverride FileInfo
	Order allow,deny
	Allow from all
	</Directory>
</VirtualHost>
{% endhighlight %}

From here I tried to browse to my new site once I restarted CF and
Apache. No love. Just a crazy blank screen. After a lot of googling I
came across forum entries suggesting that I needed to install Adobe's
Flash player instead of the free Ubuntu maintained player. This is
especially true in my case, as I am running Firefox 3.5.  
  
Here's a tip when installing Adobe Flash Player on Ubuntu Linux with
Firefox 3.5. If you have installed Firefox ina directory other than
the default location where Ubuntu installs it, your Flash installation
will appear to fail. This is because it is installing in the
**default** location, and not the one you installed. To fix this
little problem download the [TAR file](http://get.adobe.com/flashplayer/?promoid=BUIGP) from Adobe and extract it to a
location of choice. Open a terminal and change directory to this
location. From here it is important to run the *flashplayer-installer*
as root like so: *sudo ./flashplayer-installer*. This allows you to
specify the installation directory of your browser.  
  
Ok, now Illudium PU-36 is started! Woot! Oh, wait... big fat error. It
cannot seem to find components in the cfcgenerator directory. At this
point I tried putting a CF mapping, but that didn't seem to help. Here's
the trick. First off, keep the mapping because it is going to come in
handy. The next task is to find the configuration file
*remoting-config.xml* in your ColdFusion installation directory's web
root. Mine is **/opt/coldfusion8/wwwroot/WEB-INF/flex**. Open this file
and find the entry for *<use-mappings>*. Set it to true, and restart
CF.  
  
Viola!! It works! Happy coding!
