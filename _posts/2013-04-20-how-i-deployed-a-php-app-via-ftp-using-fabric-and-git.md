---
layout: post
title: How I Deployed a PHP App via FTP Using Fabric and Git
date: 2013-04-20 20:58:00
author: Adam Presley
status: Published
tags: php python git
slug: how-i-deployed-a-php-app-via-ftp-using-fabric-and-git
---
A project I've been working with for a bit now uses a popular hosting
provider to host a PHP application I'm writing. My normal workflow with
this project is to make changes, commit my code to my Git repository,
push it to Github (a private account), then FTP the files manually to
the hosting provider. This works fine most of the time though I'll admit
that there are moments when I forget a file or two. Of course the
application blows up and I have to view my Git log to see what files I
may have missed when uploading the code.

<!-- excerpt -->

Today I decided to do something about this little problem. Having
recently discovered Fabric, the deployment framework and library for
Python, I started crafting a script to automate the tasks of deployment.
There are a number of ways to approach this, but for now I decided that
I have only a couple of simple requirements.

* Need to upload files from my latest commit (HEAD)
* A single command to do the upload
* Upload is done via FTP

Based on these requirements I needed to learn two additional things: how
to use FTP in Python, and how to get a log from Git in Python. I won't
go into great deal on how all this works but I will cover the basics.
First I needed to be able to connect to an FTP server. Python has a
library called [ftplib](http://docs.python.org/2/library/ftplib.html) which offers basic, fairly low-level FTP
methods. Out of the gate the first method I write looks like this.

{% highlight python %}
def _connectToFTP():
   print green("** Connecting to the server **")

   ftp = FTP(host=FTP_ADDRESS, user=FTP_USER, passwd=FTP_PASSWORD)
   return ftp
{% endhighlight %}

As you can see the FTP class is simple enough to use. Provide an
address, user name, and password and you are connected to an FTP server
and are given an object to perform further tasks with. Naturally the
next task would be to upload files, but before we can do that we have to
know what we are uploading. This would be defined as one of the
requirements listed above where I wanted to upload the files from my
last commit.

To satisfy the requirement for uploading files from my last commit I
installed the library [gitpython](https://github.com/gitpython-developers/GitPython). GitPython, simply put, is a
library used to allow your Python code to interact with Git
repositories. Here I must admit that I had some struggles as some of the
abstractions I failed to comprehend. As a result I had a hard time
getting the results I wanted just using the pure library, so I ended up
using their utility class that allows you to execute arbitrary Git
commands on the command line and get the results back. This allowed me
to get a list of files for a given commit hash.

{% highlight python %}
def _gitLatestFiles():
   print green("** Connecting to Git **")

   g = Git(REPO_ROOT)
   repo = Repo(REPO_ROOT)
   headCommit = repo.head.commit

   print "Head commit revision: %s" % headCommit
   print "Message: %s" % headCommit.message

   result = g.execute(["git", "diff-tree", "--no-commit-id", "--name-only", "-r", str(headCommit)])
   files = result.split("\n")

   return _filterForValidFiles(fileList=files)

def _filterForValidFiles(fileList):
   return [f for f in fileList if f.startswith(("components/", "www/"))]
{% endhighlight %}

There are two methods here to get the results I want. The first is
**_gitLatestFiles()**. This method talks to my local Git repository and
gets the HEAD commit entry. With it I have a commit hash that I can use
to pull logs for. I then use the library's **execute()** method that
allows me to run command line Git and store the results. In my case I
wanted to run a *diff-tree* and get only the names of the files modified
for the specified commit hash. The result of the *diff-tree* is a single
string of the modified files, so I needed to split them into a list on a
newline character.

The next method you'll see is **_filterForValidFiles()**. It is called
by the **_gitLatestFiles()** method. This piece of code ensures that
the files modified are children of a specific set of directories. If
they reside outside that directory I don't want to deploy them.

The final step in this process is to put it all together. We need to
connect to the FTP server, get a list of changed files from the HEAD
revision in Git, then push those files to the server. Let's see what
that code looks like.

{% highlight python %}
from __future__ import with_statement

import fabric
import os
from fabric.api import *
from fabric.colors import green, yellow, red
from ftplib import FTP
from git import *

###############################################################################
# SECTION: Constants
###############################################################################
DATABASE = {
   "local": {
      "userName": "user",
      "password": "password"
   }
}

FTP_ADDRESS = "ftp.something.com"
FTP_USER = "user"
FTP_PASSWORD = "password"
FTP_ROOT_DIR = "/appDirectory"

REPO_ROOT = "../"


###############################################################################
# SECTION: Private methods
###############################################################################
def _connectToFTP():
   print green("** Connecting to the server **")

   ftp = FTP(host=FTP_ADDRESS, user=FTP_USER, passwd=FTP_PASSWORD)
   return ftp

def _gitLatestFiles():
   print green("** Connecting to Git **")

   g = Git(REPO_ROOT)
   repo = Repo(REPO_ROOT)
   headCommit = repo.head.commit

   print "Head commit revision: %s" % headCommit
   print "Message: %s" % headCommit.message

   result = g.execute(["git", "diff-tree", "--no-commit-id", "--name-only", "-r", str(headCommit)])
   files = result.split("\n")

   return _filterForValidFiles(fileList=files)

def _filterForValidFiles(fileList):
   return [f for f in fileList if f.startswith(("components/", "www/"))]


###############################################################################
# SECTION: Actions
###############################################################################

def uploadLatest():
   print ""
   print green("** Upload latest changes **")

   ftp = _connectToFTP()
   files = _gitLatestFiles()

   for f in files:
      print yellow("Uploading file %s" % f)
      split = os.path.split(f)

      ftp.cwd(os.path.join(FTP_ROOT_DIR, split[0]))
      ftp.storlines("STOR %s" % split[1], open(os.path.join("../", f), "r"))

   ftp.quit()
{% endhighlight %}

The last method in the code, **uploadLatest()** is the one that glues it
all together. As mentioned before it first connects to the FTP server.
Then it gets a list of files to upload from the latest Git commit. After
this we iterate over the file names and split each of them into
directory and file name. We do this so we can issues a CWD (current
working directory) command to the FTP server, and have a file name to
pass to the STOR FTP command.

The next steps actually change the directory on the FTP server to match
the path of the changed file. From here we send the STOR command by
calling the **storlines** method on the FTP library. This method first
takes the command to use, which is **STOR** followed by the name of the
file to upload. The second argument is an open file handle to the actual
file to upload. In this case we are just using the built-in Python
**open()** method to open up the file in read mode. Finally when all is
done we issue a QUIT command by calling the **quit()** method.

All of this code is saved in my */bin* folder of the project in a file
named *fabfile.py*. To run this I open up my terminal and execute **fab
uploadLatest**. If you aren't familiar with Fabric take a look at [the
Fabric site](http://docs.fabfile.org/en/1.6/) and also
[my previous blog entry](#post/2012/10/my-first-experience-with-fabric-and-amazon-ec2) on the subject.

Happy coding!
