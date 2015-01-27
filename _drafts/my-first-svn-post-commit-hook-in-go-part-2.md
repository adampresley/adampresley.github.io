---
layout: post
title: My First SVN Post-Commit Hook in Go - Part 2
date: 2015-02-01 10:00
author: Adam Presley
tags: development golang svn
comments: true
---
![Talking](/assets/adampresley/images/posts/30H.jpg)
*Image copyright [Ryan McGuire](http://www.gratisography.com/)*

*This is part two of the two part series on how I created a Subversion post-commit hook using Go, and post a message in a HipChat room. This post follows [part one]({% post_url 2015-01-25-my-first-svn-post-commit-hook-in-go-part-1 %}) by talking about how to post the message to HipChat. I did this for the teams at my day job. **Please note that certain pieces of code have been changed to protect the innocent, and opinions here do not necessarily reflect those of my employer.***

In part one of this two part series I talked about how to execute ```svnlook.exe``` on the command line using Go, then capture that output. Once captured the idea is to post a message in [HipChat](https://www.hipchat.com/) (or any other communication platform, should you choose to) to let teams know what code is going into a trunk/master/main/release branch. Please note that this code demonstrates consuming the HipChat API v1, which was current enough at the time I wrote this hook. HipChat has now published [version 2](https://www.hipchat.com/docs/apiv2) of their API and will deprecate version 1.

<!-- excerpt -->

### Getting Started

To begin with there are three real pieces of information we need to have. The first is the HipChat API endpoint URL that we will need to communicate with. If you are using HipChat's default hosted server then the URL will be **https://api.hipchat.com/v1**. If you are hosting a HipChat server in an enterprise environment behind the firewall your URL will be different, so check with your IT department.

The next thing you will need is an API token. This token can be retrieved in a few ways, but for this application we want a **Personal Access Token**. This can be retrieved from the HipChat developer site, or from your HipChat administrator if your server is hosted behind the firewall. This token will be a long hexadecimal string, and is your key to working with the HipChat API.

Finally we need a room. In my use case I wanted all commits to our trunk branch to post a message to HipChat in a specific room that all teams could access. For the sake of this discussion we will call the room **Code Review**. You will have to make this room, and I suggest making it public so all team members can access it, regardless of team. You may have stricter requirements, so YMMV.

{% highlight go %}
const AUTH_TOKEN string = "abcdefghijklmnopqrstuvwxyz1234"
const HIPCHAT_URL string = "https://api.hipchat.com/v1"
const ROOM string = "Code Review"
{% endhighlight %}

### Sending the Message

To post a message in a HipChat room you must craft an HTTP POST to the endpoint ```/rooms/message``` with at least three pieces of information.

1. message
2. from
3. room_id

The **message** is exactly what it sounds like. It is the message you want to post to a HipChat room. This can accept basic HTML, allowing you to emphasize parts of your message. The **from** parameter is simply a name of who you want to say this message is from. For this discussion I will say the name of the bot posting the message is *Commit*. And finally the **room_id** is either an ID or name of the HipChat room to post in. This is where the constant **ROOM** that we defined above comes in.

We are also going to send a couple of additional parameters.

* color
* notify

The **color** parameter allows use to color the background of the message, making it stand out from other messages. The **notify** parameter tells HipChat clients to popup the system notification telling the user that they have a message. We don't want our code commits going unnoticed!

#### Setting Up Parameters

First thing we must do is setup parameters to send along with our POST. In Go the ```net/url``` package offers a solution in the form of the ```url.Values``` structure. This structure provides a simple means to add key/value pairs suitable for sending a POST. The code below demonstrates setting this structure up with all the parameters we want to send, as discussed in the previous section. Notice that we are formatting our message with a little HTML to emphasize the important pieces of information.

{% highlight go %}
/*
 * Craft the POST parameters
 */
data := url.Values{}
data.Add("message", fmt.Sprintf("A commit was made by <strong>%s</strong> at revision <strong>%s</strong>.<br />LOG: %s", author, rev, logMessage))
data.Add("from", "Commit")
data.Add("room_id", ROOM)
data.Add("color", "green")
data.Add("notify", "1")
{% endhighlight %}

#### Prepare the HTTP Client

To send a POST to HipChat we will use the Go package and structure ```http.Client``` from the ```net/http``` package. This will let us send a POST to the HipChat API. First let's define the final URL endpoint.

{% highlight go %}
/*
 * Setup our request url to /rooms/message
 */
urlStr := fmt.Sprintf("%s/rooms/message?format=json&auth_token=%s", HIPCHAT_URL, AUTH_TOKEN)
{% endhighlight %}

#### Setup the Request and Execute

The next step is to setup two things. The first is an instance of the ```http.Client``` structure, and the second is an ```http.Request``` that will wrap up the request we wish to make. The request structure details that we want a POST to a specific URL as well as the POST parameters we just setup. We also need to specify that our POST is sending parameters as **application/x-www-form-urlencoded** and the size in bytes of those parameters. This is done by setting the headers *Content-Type* and *Content-Length*. The client then executes the request.

{% highlight go %}
/*
 * Prepare the HTTP client and request
 */
client := &http.Client{}

request, err := http.NewRequest("POST", urlStr, bytes.NewBufferString(data.Encode()))
if err != nil {
   log.Fatalf("An error occurred crafting the POST request: %s", err.Error())
}

request.Header.Add("Content-Type", "application/x-www-form-urlencoded")
request.Header.Add("Content-Length", strconv.Itoa(len(data.Encode())))

/*
 * Execute!
 */
response, err := client.Do(request)
if err != nil {
   log.Fatalf("An error occurred sending the POST request: %s", err.Error())
}
{% endhighlight %}

### The Culmination

That is quite a bit of information! To summarize we have code that retrieves author and commit information from Subversion using the ```svnlook.exe``` command line tool. With the commit information we craft an HTTP POST to the HipChat API, sending a formatted message to a room.

Below is the code for the whole thing. If you have any questions feel free to comment below, in G+, or on Twitter. Or perhaps you are interested in having something like this written for your organization. If so please contact me at [adam@adampresley.com](mailto:adam@adampresley.com) and let's talk! Cheers, and happy coding!

#### blog/adampresley/postcommit/svnlook/SvnLook.go

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

#### /blog/adampresley/postcommit/post-commit.go

{% highlight go %}
package main

import (
   "bytes"
   "fmt"
   "log"
   "net/http"
   "net/url"
   "os"
   "strconv"
   "strings"
   "regexp"

   "blog/adampresley/postcommit/svnlook"
)

const AUTH_TOKEN string = "abcdefghijklmnopqrstuvwxyz1234"
const HIPCHAT_URL string = "https://api.hipchat.com/v1"
const ROOM string = "Code Review"
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

         /*
          * Craft the POST parameters
          */
         data := url.Values{}
         data.Add("message", fmt.Sprintf("A commit was made by <strong>%s</strong> at revision <strong>%s</strong>.<br />LOG: %s", author, rev, logMessage))
         data.Add("from", "Commit")
         data.Add("room_id", ROOM)
         data.Add("color", "green")
         data.Add("notify", "1")

         /*
          * Seutp our request url to /rooms/message
          */
         urlStr := fmt.Sprintf("%s/rooms/message?format=json&auth_token=%s", HIPCHAT_URL, AUTH_TOKEN)

         /*
          * Prepare the HTTP client and request
          */
         client := &http.Client{}

         request, err := http.NewRequest("POST", urlStr, bytes.NewBufferString(data.Encode()))
         if err != nil {
            log.Fatalf("An error occurred crafting the POST request: %s", err.Error())
         }

         request.Header.Add("Content-Type", "application/x-www-form-urlencoded")
         request.Header.Add("Content-Length", strconv.Itoa(len(data.Encode())))

         /*
          * Execute!
          */
         response, err := client.Do(request)
         if err != nil {
            log.Fatalf("An error occurred sending the POST request: %s", err.Error())
         }

         log.Println(response.Status)
      }
   } else {
      log.Println("Not enough arguments to activate svnPostCommitHook power!")
   }
}
{% endhighlight %}
