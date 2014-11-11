---
layout: post
title: "Sublime Plugin: Compare"
date: 2011-11-08 19:51:00
author: Adam Presley
status: Published
tags: python sublime
slug: sublime-plugin-compare
---

Not one to be completely happy with all existing features I found the
diff tool in Sublime Text 2 to be... lacking. The built in diff tool
allows you to select two files in the folder explorer, right click and
choose *Diff Files*. This action will produce a unified diff output.

I, however, would like to have a way to use a graphical diff tool,
because I'm way too lazy to read the unified diff. So I decided to write
a plugin for Sublime. This plugin will allow me to map a key to run a
command that will compare the current tab's content with the contents of
the tab to the right. Let's walk through what that looks like. By the
way, plugins are written in Python.

First let's get the boilerplate stuff out of the way. We'll need to
import some stuff and create a class that extends a built in Sublime
class that will allow us to create a plugin.

{% highlight python %}
import sublime, sublime_pluginimport subprocessfrom subprocess
import Popen
from string import Template

class CompareCommand(sublime_plugin.TextCommand):
{% endhighlight %}

Now we need to define the **run()**method. The first argument is a
reference to *self*, followed by the edit environment, and an argument
called *command*, which will be an string containing the path to the
diff tool to run, and placeholders for the left and right file names.
More on that later when we map our new command to a key.

{% highlight python %}
	def run(self, edit, command):
{% endhighlight %}

The first task is to write two methods. The first will identify the
current buffer, or tab, we are working on when our plugin executes. The
second method will identify the buffer directly to the right. These
files will make up the left and right files to send to a diff tool.This
task is done by first getting the current view's ID, then looping over
all open views and finding the tab adjacent to our open one.

{% highlight python %}
	def __getCurrentBufferInfo(self):
		return {
			"id": self.view.buffer_id(),
			"fileName": self.view.file_name()
		}

	def __getNextBufferInfo(self, currentBuffer):
		result = None
		found = False

		for b in self.view.window().views():
			if found:
				result = {
					"id": b.buffer_id(),
					"fileName": b.file_name()
				}

				break

			if b.buffer_id() == currentBuffer["id"]:
				found = True

		return result
{% endhighlight %}

Once we have the two file names we are going to compare let's open up
the specified diff tool and pass the files in. Our command in the key
mapping will define a string, and there will be two variables in there:
**$leftFile**and **$rightFile**. To fill in the variables we will use
the **Template**class. Once our string is filled in with file names we
will use the **Subprocess**class to execute. If there are any errors
they will be output to the Sublime console. Here is the entire listing.

{% highlight python %}
import sublime, sublime_plugin
import subprocess
from subprocess import Popen
from string import Template

class CompareCommand(sublime_plugin.TextCommand):
	def run(self, edit, command):
		currentBuffer = self.__getCurrentBufferInfo()
		nextBuffer = self.__getNextBufferInfo(currentBuffer)

		#
		# Compare the contents of the current buffer with the
		# contents of the next buffer (if there is one).
		#
		processCommand = []

		if nextBuffer != None:
			processCommand = Template(command).substitute(leftFile = currentBuffer["fileName"], rightFile = nextBuffer["fileName"])
			print processCommand

			#
			# Execute the command. Capture output on STDOUT, and write it out
			# to Sublime's console.
			#
			p = Popen(processCommand, shell = True, stdout = subprocess.PIPE, stderr = subprocess.STDOUT)
			for line in p.stdout.readlines():
				print line

			ret = p.wait()
			print "Return code: %s" % (ret)

	def __getCurrentBufferInfo(self):
		return {
			"id": self.view.buffer_id(),
			"fileName": self.view.file_name()
		}

	def __getNextBufferInfo(self, currentBuffer):
		result = None
		found = False

		for b in self.view.window().views():
			if found:
				result = {
					"id": b.buffer_id(),
					"fileName": b.file_name()
				}

				break

			if b.buffer_id() == currentBuffer["id"]:
				found = True

		return result
{% endhighlight %}

And now we can define a key mapping to be able to call our nifty plugin.
Here is an example of a key mapping using the Meld diff tool in Ubuntu.

{% highlight javascript %}
{ "keys": [ "ctrl+shift+c" ], "command": "compare", "args": { "command": "/usr/bin/meld '$leftFile' '$rightFile'" } }
{% endhighlight %}

If you take the code above and save it in the Packages/User directory as
**CompareCommand.py**, and modify your user key mappings as shown above,
you should be able to hit CTRL+SHIFT+C and compare two files.

Not perfect, but was fun to write, and will prove useful for me until
the folks at Sublime put a full feature in and replace the need for this
plugin. Happy coding!
