---
layout: post
title: My First SVN Post-Commit Hook in Go - Part 1
date: 2015-01-25 13:32
author: Adam Presley
tags: development golang svn
comments: true
---
![Man with headphones](/assets/adampresley/images/posts/90H.jpg)
*Image copyright [Ryan McGuire](http://www.gratisography.com/)*

*This is part one of a two part series on how I created a Subversion post-commit hook using Go, and post a message in a HipChat room. I did this for the teams at my day job. **Please note that certain pieces of code have been changed to protect the innocent, and opinions here do not necessarily reflect those of my employer.***

<!-- excerpt -->

There has been an ongoing effort with the various teams I work with to improve code review. To provide more visibility to code being committed to our trunk branch in Subversion I decided a post-commit hook could help us achieve that visibility. This post-commit hook would send a message to a HipChat room, dedicated to reviewing code changes and information about each commit. This would allow teams to know when changes were made to our trunk branch and have the chance to review the code across teams. This strategy has been an effective one in reducing errors in code and improving overall code quality. This post will deal with utilizing Subversion to retrieve information about a commit. This information is what will be used in part 2 to post a message in HipChat.

When you commit code into Subversion a series of events occur. The final event, and the one we are most interested in for this exercise, is the **post-commit**. Once a commit is complete and recorded to your Subversion repository Subversion will execute a post-commit handler if one is defined. This is usually setup in the form of a shell script or executable. When Subversion executes the post-commit handler it provides two pieces of useful information: ```repository path``` and the ```revision number```. Armed with this we can query Subversion for more information about the commit, such as who committed the work, and the log message that goes along with it. So the following code will demonstrate how we can get all this information. Note that this code isn't the prettiest and could use some improvement, and it also requires that you have the SVN command line tools installed at a specific location.

### Command Line Arguments

The first order of business is to get the repository path and revision number. This can be done using the ```os``` package, which provides an array of command line arguments called **Args**. The very first argument will be the executable Go program, so we are expecting the repository path and revision number at positions 1 and 2, respectively. Thus means we are expecting at least 3 arguments, and we are interested in the first two (past the executable name).

{% highlight go %}
if len(os.Args) >= 3 {
	repo := os.Args[1]
	rev := os.Args[2]
	log.Printf("Commit revision %s against %s", rev, repo)
{% endhighlight %}

### Getting Info from Subversion

Once we have the repository path and revision number we need to get author and log message information from Subversion. This is where we will make a new file to put a few functions into. These functions will do the talking to the command line to execute Subversion commands. More specifically we will use the ```svnlook.exe``` program to get what we need. Let's create a folder named ```svnLook``` and make a file called **SvnLook.go**. This will house functions to communicate with SVN.

#### Give It a Home

There are several operations that we are going to want to perform. We will want to get the author, log, and changed paths for a specific commit. As such there is some information that we can use across all our functions. For this we can setup a structure, and then attach functions to it.

{% highlight go %}
type SvnLook struct {
   SvnLookBinaryPath string
   RepositoryPath    string
   Revision          string
}
{% endhighlight %}

It would also be handy to have a function to return a new ```SvnLook``` structure instance. So let's make one. It doesn't need to do much. Just create the instance of the ```SvnLook``` structure, replace backslashes with forward slashes, and return it.

{% highlight go %}
func NewSvnLook(svnLookBinaryPath, repositoryPath, revision string) *SvnLook {
   return &SvnLook{
      SvnLookBinaryPath: svnLookBinaryPath,
      RepositoryPath: strings.Replace(repositoryPath, "\\", "/", -1),
      Revision: revision,
   }
}
{% endhighlight %}

#### Executing Commands

Now that we have a place to put Subversion commands we need to actually execute them. There are three commands we are going to need, each switches on the ```svnlook.exe``` program. With that in mind let's make a function that executes the ```svnlook.exe``` program which takes in arguments and returns any output spit out. To execute a command line program we'll make use of the ```os/exec``` Go package. This will take an argument of the specific command we want to execute It then calls **exec.Command()**. Each Subversion command we execute will write information to the console normally, but we need that information. To capture this we redirect the **Stdout** to a byte array buffer. After running the command that byte array will contain the console output, which we return back as a string.

{% highlight go %}
func (this *SvnLook) executeCommand(command string) (string, error) {
   var out bytes.Buffer

   cmd := exec.Command(this.SvnLookBinaryPath, command, this.RepositoryPath, "--revision", this.Revision)
   cmd.Stdout = &out

   err := cmd.Run()
   if err != nil {
      return "", err
   }

   return out.String(), nil
}
{% endhighlight %}

#### The Author

Now that we have a function to run the ```svnlook.exe``` program with an arbitrary command it should be trivial to get the data we want. One of the things we'd like to know is the author of the commit. On the console that would look like ```svnlook.exe author /repo/path --revision 12345```. Our **executeCommand()** function takes care of most of that, so we'll just need to tell it that we want to get author information.

{% highlight go %}
func (this *SvnLook) GetAuthor() (string, error) {
   result, err := this.executeCommand("author")
   return strings.TrimSpace(result), err
}
{% endhighlight %}

#### Directories Changed

We also need to know what directory paths were affected by this commit. On the console that would look like ```svnlook.exe dirs-changed /repo/path --revision 12345```. The only thing we need to add here is to take the result and split it by newline. This is because each affected directory of the commit is on a separate line.

{% highlight go %}
func (this *SvnLook) GetDirectoriesChanged() ([]string, error) {
   result, err := this.executeCommand("dirs-changed")
   if err != nil {
      return make([]string, 0), err
   }

   return strings.Split(result, "\n"), nil
}
{% endhighlight %}

#### Log Message

Hopefully each commit a developer makes is accompanied by a detailed log message. We'd like to see that too. On the console that would look like ```svnlookexe log /repo/path --revision 12345```.

{% highlight go %}
func (this *SvnLook) GetLog() (string, error) {
   return this.executeCommand("log")
}
{% endhighlight %}

### Making Good Use

Now let's make good use of our new functions. When we last left our main program was getting repository path and revision information. This is exactly what we need to make use of our new ```SvnLook``` functions. That, and a path to the executable. So let's put them to use!

#### The Right Change?

The first order of business is to create an instance and get affected paths. We want to know what paths were affected because in our situation we only want to alert HipChat if a commit was made against the trunk branch. Other branches we do not care about.

{% highlight go %}
svnLook := svnlook.NewSvnLook(SVNLOOK_PATH, repo, rev)

affectedPaths, err := svnLook.GetDirectoriesChanged()
if err != nil || len(affectedPaths) <= 0 {
   log.Fatalf("An error occurred running svnlook: %s", err.Error())
}

if strings.Contains(strings.Join(affectedPaths, " "), "trunk/") {
}
{% endhighlight %}

#### Who Did What?

Now that we know what paths changed, and if we care about them, let's get the good stuff. The author and log message are what ultimately will be important to convey in a HipChat message.

{% highlight go %}
author, err := svnLook.GetAuthor()
if err != nil {
   log.Println("ERROR - No author information available")
}

logMessage, err := svnLook.GetLog()
if err != nil {
   log.Println("ERROR - No log information available")
}
{% endhighlight %}

### The Culmination

Let's see what it all looks like so far. This isn't everything, because the rest will be done in part 2, but for the moment I'll put what we have right here.

#### svnlook/SvnLook.go

{% highlight go %}
package svnlook

import (
   "bytes"
   "os/exec"
   "strings"
)

type SvnLook struct {
   SvnLookBinaryPath string
   RepositoryPath    string
   Revision          string
}

func NewSvnLook(svnLookBinaryPath, repositoryPath, revision string) *SvnLook {
   return &SvnLook{
      SvnLookBinaryPath: svnLookBinaryPath,
      RepositoryPath: strings.Replace(repositoryPath, "\\", "/", -1),
      Revision: revision,
   }
}

func (this *SvnLook) executeCommand(command string) (string, error) {
   var out bytes.Buffer

   cmd := exec.Command(this.SvnLookBinaryPath, command, this.RepositoryPath, "--revision", this.Revision)
   cmd.Stdout = &out

   err := cmd.Run()
   if err != nil {
      return "", err
   }

   return out.String(), nil
}

func (this *SvnLook) GetAuthor() (string, error) {
   result, err := this.executeCommand("author")
   return strings.TrimSpace(result), err
}

func (this *SvnLook) GetDirectoriesChanged() ([]string, error) {
   result, err := this.executeCommand("dirs-changed")
   if err != nil {
      return make([]string, 0), err
   }

   return strings.Split(result, "\n"), nil
}

func (this *SvnLook) GetLog() (string, error) {
   return this.executeCommand("log")
}
{% endhighlight %}

#### post-commit.go

{% highlight go %}
package main

import (
   "log"
   "os"
   "strings"
   "regexp"
)

const SVNLOOK_PATH string = "C:\\progra~1\\Subversion\\bin\\svnlook.exe"

func main() {
   log.Printf("svnPostCommitHook activated with %d arguments", len(os.Args))

   if len(os.Args) >= 3 {
      repo := os.Args[1]
      rev := os.Args[2]
      log.Printf("Commit revision %s against %s", rev, repo)

      svnLook := svnlook.NewSvnLook(SVNLOOK_PATH, repo, rev)

      affectedPaths, err := svnLook.GetDirectoriesChanged()
      if err != nil || len(affectedPaths) <= 0 {
         log.Fatalf("An error occurred running svnlook: %s", err.Error())
      }

      /*
       * Makes sure the repo name contains /trunk/. If so get
       * author and log message.
       */
      if strings.Contains(strings.Join(affectedPaths, " "), "trunk/") {
         author, err := svnLook.GetAuthor()
         if err != nil {
            log.Println("ERROR - No author information available")
         }

         logMessage, err := svnLook.GetLog()
         if err != nil {
            log.Println("ERROR - No log information available")
         }
      }
   } else {
      log.Println("Not enough arguments to activate svnPostCommitHook power!")
   }
}
{% endhighlight %}

Until the next installment, happy coding!